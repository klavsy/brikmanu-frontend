<!-- frontend/src/components/PassportCard.vue -->
<template>
  <div class="relative group">
    <!-- Glassmorphism Card -->
    <div :class="['p-6 rounded-3xl backdrop-blur-xl border-2 transition-all duration-500 shadow-2xl', 
                 user.isVIP ? 'bg-gradient-to-br from-yellow-100 to-white border-yellow-500' : 'bg-white/80 border-white']">
      
      <div class="flex justify-between items-start mb-8">
        <div class="text-left">
          <p class="text-xs font-bold text-gray-500 uppercase tracking-widest">Dzimtas Pase</p>
          <h2 class="text-3xl font-black text-[#9E1B32]">{{ user.name }}</h2>
        </div>
        <div class="bg-[#9E1B32] text-white p-2 rounded-lg text-xs font-bold">
          Brikmaņi
        </div>
      </div>

      <div class="flex justify-center mb-8">
        <div class="w-32 h-32 bg-gray-200 rounded-full border-4 border-white shadow-inner flex items-center justify-center text-4xl font-bold text-[#9E1B32]">
          {{ user.name[0] }}
        </div>
      </div>

      <div class="text-center mb-6">
        <p class="text-sm text-gray-600">Atzalas zariņš: <span class="font-bold text-gray-800">{{ user.branch }}</span></p>
      </div>

      <!-- Mock QR Code -->
      <div class="flex justify-center p-4 bg-white rounded-2xl shadow-inner mb-6">
        <img src="https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=BrikmanuPassport" alt="QR" class="w-32 h-32" />
      </div>

      <button @click="$emit('go-to-game')" 
              class="w-full py-4 bg-slate-900 text-white rounded-2xl font-bold hover:scale-105 transition-transform">
        Iesest stīgoties (Join Game)
      </button>
    </div>
    
    <div v-if="user.isVIP" class="absolute -top-4 -right-4 bg-yellow-500 text-white px-4 py-1 rounded-full text-xs font-black shadow-lg animate-bounce">
      VIP STATUSS
    </div>
  </div>
</template>

<script setup>
defineProps(['user']);
</script>

