<template>
  <div class="min-h-screen bg-[#F5F7FA] pb-nav">

    <!-- Header -->
    <div class="relative bg-gradient-to-br from-brand-700 to-brand-800 px-5 pt-12 pb-20 overflow-hidden">
      <div class="absolute -top-6 -right-6 w-32 h-32 rounded-full bg-white/5" />
      <div class="absolute bottom-2 left-4 w-16 h-16 rounded-full bg-white/5" />
      <div class="relative flex items-center gap-3">
        <button @click="$router.back()" class="w-8 h-8 rounded-full bg-white/10 flex items-center justify-center">
          <ChevronLeftIcon class="w-5 h-5 text-white" />
        </button>
        <div>
          <p class="text-white/60 text-xs font-medium">Metode Pembayaran</p>
          <h1 class="text-white font-bold text-lg">Bayar Tagihan</h1>
        </div>
      </div>
    </div>

    <div class="relative z-10 px-4 -mt-10 space-y-4">

      <!-- QRIS Section -->
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100/80 overflow-hidden">
        <div class="flex items-center gap-3 p-4 border-b border-gray-50">
          <div class="w-9 h-9 rounded-xl bg-violet-50 flex items-center justify-center">
            <QrCodeIcon class="w-5 h-5 text-violet-600" />
          </div>
          <div>
            <p class="text-gray-800 font-bold text-sm">QRIS</p>
            <p class="text-gray-400 text-xs">Scan & bayar dari semua e-wallet</p>
          </div>
          <span class="ml-auto text-[10px] bg-emerald-100 text-emerald-700 font-bold px-2 py-1 rounded-full">AKTIF</span>
        </div>
        <div class="p-5 flex flex-col items-center">
          <div class="w-48 h-48 bg-gray-50 rounded-2xl border-2 border-dashed border-gray-200 flex items-center justify-center mb-3">
            <div class="text-center text-gray-300">
              <QrCodeIcon class="w-16 h-16 mx-auto mb-2" />
              <p class="text-xs">QRIS tersedia</p>
              <p class="text-xs">di kasir pesantren</p>
            </div>
          </div>
          <p class="text-xs text-gray-400 text-center">Scan QR di kasir pesantren menggunakan GoPay, OVO, Dana, ShopeePay, atau m-Banking manapun</p>
        </div>
      </div>

      <!-- Virtual Account -->
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100/80 overflow-hidden">
        <div class="flex items-center gap-3 p-4 border-b border-gray-50">
          <div class="w-9 h-9 rounded-xl bg-blue-50 flex items-center justify-center">
            <BuildingLibraryIcon class="w-5 h-5 text-blue-600" />
          </div>
          <div>
            <p class="text-gray-800 font-bold text-sm">Virtual Account</p>
            <p class="text-gray-400 text-xs">Transfer bank ke nomor VA pesantren</p>
          </div>
        </div>
        <div class="p-4 space-y-3">
          <div v-for="bank in virtualAccounts" :key="bank.bank"
               class="flex items-center justify-between p-3 bg-gray-50 rounded-xl">
            <div class="flex items-center gap-3">
              <div class="w-10 h-10 rounded-lg bg-white shadow-sm flex items-center justify-center">
                <span class="text-xs font-black text-gray-700">{{ bank.bank }}</span>
              </div>
              <div>
                <p class="text-gray-700 font-semibold text-sm">{{ bank.nama }}</p>
                <p class="text-gray-400 text-xs font-mono">{{ bank.nomor }}</p>
              </div>
            </div>
            <button @click="copyVA(bank.nomor)"
                    class="p-2 rounded-lg bg-white shadow-sm active:bg-gray-100">
              <ClipboardDocumentIcon class="w-4 h-4 text-gray-500" />
            </button>
          </div>
        </div>
      </div>

      <!-- E-Money / E-Wallet -->
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100/80 overflow-hidden">
        <div class="flex items-center gap-3 p-4 border-b border-gray-50">
          <div class="w-9 h-9 rounded-xl bg-amber-50 flex items-center justify-center">
            <DevicePhoneMobileIcon class="w-5 h-5 text-amber-600" />
          </div>
          <div>
            <p class="text-gray-800 font-bold text-sm">E-Wallet & E-Money</p>
            <p class="text-gray-400 text-xs">Transfer langsung ke rekening pesantren</p>
          </div>
        </div>
        <div class="p-4 grid grid-cols-3 gap-3">
          <div v-for="ew in ewallets" :key="ew.name"
               class="flex flex-col items-center gap-2 p-3 bg-gray-50 rounded-xl">
            <div class="w-10 h-10 rounded-xl flex items-center justify-center text-xl"
                 :class="ew.bg">{{ ew.emoji }}</div>
            <p class="text-xs font-semibold text-gray-700 text-center">{{ ew.name }}</p>
            <p class="text-[10px] text-gray-400 font-mono text-center">{{ ew.nomor }}</p>
          </div>
        </div>
      </div>

      <!-- Info Box -->
      <div class="bg-amber-50 rounded-2xl border border-amber-100 p-4 flex gap-3">
        <InformationCircleIcon class="w-5 h-5 text-amber-500 flex-shrink-0 mt-0.5" />
        <div>
          <p class="text-amber-800 font-semibold text-sm mb-1">Cara Pembayaran</p>
          <ol class="text-amber-700 text-xs space-y-1 list-decimal list-inside leading-relaxed">
            <li>Pilih metode pembayaran di atas</li>
            <li>Transfer sesuai nominal tagihan</li>
            <li>Upload bukti transfer di menu Tagihan</li>
            <li>Admin akan memverifikasi dalam 1x24 jam</li>
          </ol>
        </div>
      </div>

    </div>

    <!-- Copy Toast -->
    <Transition name="fade">
      <div v-if="copied"
           class="fixed bottom-24 left-1/2 -translate-x-1/2 bg-gray-800 text-white text-sm px-4 py-2.5 rounded-full shadow-lg z-50">
        ✓ Nomor disalin
      </div>
    </Transition>

  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {
  ChevronLeftIcon, QrCodeIcon, BuildingLibraryIcon,
  DevicePhoneMobileIcon, ClipboardDocumentIcon, InformationCircleIcon,
} from '@heroicons/vue/24/outline'

const copied = ref(false)

const virtualAccounts = [
  { bank: 'BCA',   nama: 'Bank Central Asia',  nomor: '1234567890' },
  { bank: 'BNI',   nama: 'Bank Negara Indonesia', nomor: '9876543210' },
  { bank: 'BSI',   nama: 'Bank Syariah Indonesia', nomor: '7123456789' },
  { bank: 'MANDIRI', nama: 'Bank Mandiri', nomor: '1400012345678' },
]

const ewallets = [
  { name: 'GoPay',     emoji: '💚', bg: 'bg-green-50',   nomor: '08xx-xxxx-xxxx' },
  { name: 'OVO',       emoji: '💜', bg: 'bg-purple-50',  nomor: '08xx-xxxx-xxxx' },
  { name: 'Dana',      emoji: '💙', bg: 'bg-blue-50',    nomor: '08xx-xxxx-xxxx' },
  { name: 'ShopeePay', emoji: '🧡', bg: 'bg-orange-50',  nomor: '08xx-xxxx-xxxx' },
  { name: 'LinkAja',   emoji: '❤️',  bg: 'bg-red-50',     nomor: '08xx-xxxx-xxxx' },
  { name: 'Qris',      emoji: '🔳', bg: 'bg-gray-50',    nomor: 'Scan di kasir' },
]

function copyVA(nomor: string) {
  navigator.clipboard.writeText(nomor).catch(() => {})
  copied.value = true
  setTimeout(() => { copied.value = false }, 2000)
}
</script>
