<template>
  <div class="min-h-screen bg-[#F5F7FA] pb-nav">

    <!-- Header -->
    <div class="relative bg-gradient-to-br from-brand-700 to-brand-800 px-5 pt-12 pb-28 overflow-hidden">
      <div class="absolute -top-6 -right-6 w-32 h-32 rounded-full bg-white/5" />
      <div class="absolute bottom-2 left-4 w-16 h-16 rounded-full bg-white/5" />
      <div class="relative flex items-center gap-3">
        <button @click="$router.back()" class="w-8 h-8 rounded-full bg-white/10 flex items-center justify-center active:bg-white/20">
          <ChevronLeftIcon class="w-5 h-5 text-white" />
        </button>
        <div>
          <p class="text-white/60 text-xs font-medium">Tabungan Santri</p>
          <h1 class="text-white font-bold text-lg">{{ activeSantri?.nama ?? '—' }}</h1>
        </div>
      </div>
    </div>

    <!-- Virtual Card -->
    <div class="relative z-10 px-4 -mt-20 mb-5">
      <div class="relative bg-gradient-to-br from-brand-600 via-brand-700 to-brand-900 rounded-3xl p-6 shadow-2xl shadow-brand-900/30 overflow-hidden">
        <!-- Card decoration -->
        <div class="absolute -top-8 -right-8 w-36 h-36 rounded-full bg-white/5" />
        <div class="absolute bottom-0 right-8 w-24 h-24 rounded-full bg-white/5" />
        <div class="absolute top-4 right-4">
          <div class="flex gap-1">
            <div class="w-6 h-6 rounded-full bg-amber-400/80" />
            <div class="w-6 h-6 rounded-full bg-amber-300/50 -ml-3" />
          </div>
        </div>

        <p class="text-white/60 text-xs font-medium mb-1">Saldo Tabungan</p>
        <template v-if="loading">
          <div class="skeleton h-8 w-44 rounded mb-4 bg-white/20" />
        </template>
        <template v-else>
          <p class="text-white text-3xl font-black mb-4">
            Rp {{ (tabungan?.saldo ?? 0).toLocaleString('id-ID') }}
          </p>
        </template>

        <div class="h-px bg-white/10 mb-4" />

        <div class="flex justify-between items-center">
          <div>
            <p class="text-white/50 text-[10px] mb-0.5">Nama Santri</p>
            <p class="text-white text-sm font-bold">{{ activeSantri?.nama ?? '—' }}</p>
          </div>
          <div class="text-right">
            <p class="text-white/50 text-[10px] mb-0.5">No. Rekening</p>
            <p class="text-white/80 text-sm font-mono">{{ tabungan?.no_rekening ?? '—' }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Limit Jajan Harian -->
    <div class="px-4 mb-4">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100/80 p-5">
        <div class="flex items-center justify-between mb-3">
          <div>
            <p class="text-gray-800 font-bold text-sm">Limit Jajan Harian</p>
            <p class="text-gray-400 text-xs mt-0.5">Reset setiap tengah malam</p>
          </div>
          <button @click="openLimitSheet"
                  class="text-brand-700 text-xs font-bold px-3 py-1.5 rounded-lg bg-brand-50 active:bg-brand-100">
            Atur
          </button>
        </div>

        <template v-if="loading">
          <div class="skeleton h-3 w-full rounded mb-2" />
          <div class="skeleton h-2 w-28 rounded" />
        </template>
        <template v-else>
          <div class="h-2 bg-gray-100 rounded-full overflow-hidden mb-2">
            <div class="h-full rounded-full transition-all duration-500"
                 :class="progressColor"
                 :style="{ width: limitProgress + '%' }" />
          </div>
          <div class="flex justify-between text-xs text-gray-400">
            <span>Terpakai: Rp {{ (tabungan?.terpakai_hari_ini ?? 0).toLocaleString('id-ID') }}</span>
            <span>Limit: Rp {{ (tabungan?.limit_harian ?? 0).toLocaleString('id-ID') }}</span>
          </div>
        </template>
      </div>
    </div>

    <!-- Stats Row -->
    <div class="px-4 mb-4 grid grid-cols-2 gap-3">
      <div class="bg-white rounded-2xl p-4 border border-gray-100/80 shadow-sm">
        <p class="text-gray-400 text-xs mb-1">Pemasukan Bulan Ini</p>
        <p class="text-emerald-600 font-black text-base">
          Rp {{ (tabungan?.pemasukan_bulan ?? 0).toLocaleString('id-ID') }}
        </p>
      </div>
      <div class="bg-white rounded-2xl p-4 border border-gray-100/80 shadow-sm">
        <p class="text-gray-400 text-xs mb-1">Pengeluaran Bulan Ini</p>
        <p class="text-red-500 font-black text-base">
          Rp {{ (tabungan?.pengeluaran_bulan ?? 0).toLocaleString('id-ID') }}
        </p>
      </div>
    </div>

    <!-- Freeze Toggle -->
    <div class="px-4 mb-4">
      <div class="bg-white rounded-2xl p-4 border border-gray-100/80 shadow-sm flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="w-9 h-9 rounded-xl bg-blue-50 flex items-center justify-center">
            <LockClosedIcon class="w-4 h-4 text-blue-600" />
          </div>
          <div>
            <p class="text-gray-800 font-semibold text-sm">Bekukan Akun</p>
            <p class="text-gray-400 text-xs">Nonaktifkan sementara transaksi</p>
          </div>
        </div>
        <button @click="toggleFreeze"
                class="w-12 h-6 rounded-full transition-colors duration-200 flex items-center px-0.5"
                :class="tabungan?.frozen ? 'bg-blue-600' : 'bg-gray-200'">
          <div class="w-5 h-5 rounded-full bg-white shadow transition-transform duration-200"
               :class="tabungan?.frozen ? 'translate-x-6' : 'translate-x-0'" />
        </button>
      </div>
    </div>

    <!-- Backdrop -->
    <Transition name="fade">
      <div v-if="sheet" class="fixed inset-0 bg-black/50 z-[60]" @click="sheet = false" />
    </Transition>

    <!-- Limit Sheet -->
    <Transition name="sheet-up">
      <div v-if="sheet"
           class="fixed bottom-0 left-0 right-0 z-[70] bg-white rounded-t-3xl shadow-2xl px-5 pt-5"
           :style="{ paddingBottom: 'calc(1.5rem + env(safe-area-inset-bottom, 0px))' }">
        <div class="w-10 h-1 bg-gray-200 rounded-full mx-auto mb-5" />
        <h3 class="text-gray-800 font-bold text-base mb-1">Atur Limit Jajan Harian</h3>
        <p class="text-gray-400 text-sm mb-5">Masukkan nominal limit pengeluaran per hari</p>

        <div class="relative mb-4">
          <span class="absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 font-medium">Rp</span>
          <input v-model="newLimit" type="number" inputmode="numeric"
                 placeholder="50.000"
                 class="w-full pl-10 pr-4 py-3.5 rounded-xl border border-gray-200 text-gray-800
                        text-base focus:outline-none focus:ring-2 focus:ring-brand-500/30
                        focus:border-brand-400 bg-gray-50" />
        </div>

        <div class="flex gap-3">
          <button @click="sheet = false"
                  class="flex-1 py-3.5 rounded-2xl border border-gray-200 text-gray-700 font-semibold">
            Batal
          </button>
          <button @click="saveLimit" :disabled="savingLimit"
                  class="flex-1 py-3.5 rounded-2xl bg-brand-700 text-white font-bold
                         disabled:opacity-60 active:bg-brand-800">
            {{ savingLimit ? 'Menyimpan...' : 'Simpan' }}
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useSantriStore } from '@/stores/santri'
import { waliApi } from '@/api/wali'
import { ChevronLeftIcon, LockClosedIcon } from '@heroicons/vue/24/outline'

const santriStore  = useSantriStore()
const activeSantri = computed(() => santriStore.active)
const loading      = ref(true)
const tabungan     = ref<any>(null)
const sheet        = ref(false)
const newLimit     = ref('')
const savingLimit  = ref(false)

const limitProgress = computed(() => {
  if (!tabungan.value?.limit_harian) return 0
  return Math.min(100, (tabungan.value.terpakai_hari_ini / tabungan.value.limit_harian) * 100)
})

const progressColor = computed(() => {
  const p = limitProgress.value
  if (p >= 90) return 'bg-red-500'
  if (p >= 60) return 'bg-amber-400'
  return 'bg-emerald-500'
})

function openLimitSheet() {
  newLimit.value = tabungan.value?.limit_harian?.toString() ?? ''
  sheet.value = true
}

async function saveLimit() {
  if (!activeSantri.value || !newLimit.value) return
  savingLimit.value = true
  try {
    await waliApi.setLimitJajan(activeSantri.value.id, parseInt(newLimit.value))
    if (tabungan.value) tabungan.value.limit_harian = parseInt(newLimit.value)
    sheet.value = false
  } catch { /* handle */ } finally { savingLimit.value = false }
}

async function toggleFreeze() {
  if (!activeSantri.value || !tabungan.value) return
  const newState = !tabungan.value.frozen
  tabungan.value.frozen = newState
  try { await waliApi.freezeTabungan(activeSantri.value.id, newState) }
  catch { tabungan.value.frozen = !newState }
}

onMounted(async () => {
  if (!activeSantri.value) return
  try { tabungan.value = await waliApi.tabungan(activeSantri.value.id) }
  finally { loading.value = false }
})
</script>
