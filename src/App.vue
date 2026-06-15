<template>
  <div class="min-h-screen bg-slate-50 font-sans text-slate-900 transition-all duration-500">
    <!-- TOP NAVIGATION -->
    <nav class="bg-[#9E1B32] p-4 text-white flex justify-between items-center shadow-md sticky top-0 z-50">
      <div class="flex items-center gap-2">
        <span class="text-2xl">🐾</span>
        <h1 class="text-lg font-black tracking-tighter uppercase">Brikmaņu Dzimta 2024</h1>
      </div>
      <div v-if="user" class="text-xs bg-white text-[#9E1B32] px-3 py-1 rounded-full font-bold shadow-sm">
        {{ user.name }}
      </div>
    </nav>

    <main class="p-4 max-w-2xl mx-auto">
      <!-- WELCOME / LOGIN SCREEN -->
      <div v-if="!user && view !== 'admin'" class="flex flex-col items-center justify-center mt-12 text-center">
        <div class="relative mb-6">
          <div class="w-32 h-32 bg-yellow-100 rounded-full flex items-center justify-center text-6xl shadow-inner animate-bounce">🐾</div>
          <div class="absolute -top-2 -right-2 bg-red-500 text-white text-[10px] px-2 py-1 rounded-full font-bold uppercase">Princess</div>
        </div>
        
        <h2 class="text-4xl font-black text-slate-800 mb-2 leading-tight">Sagaidām jūs <br/><span class="text-[#9E1B32]">Brikmaņu Odyssey</span></h2>
        <p class="text-slate-500 mb-8 px-6">Skanē pases kodu vai ievadi ID, lai pievienotos ģimenes sapulcionšanai.</p>

        <div class="w-full max-w-sm bg-white p-8 rounded-3xl shadow-2xl border-b-4 border-[#9E1B32]">
          <input v-model="passportId" placeholder="Ievadi pases ID (piem. B001)" 
                 class="w-full p-4 rounded-2xl border-2 border-slate-200 mb-4 outline-none focus:border-[#9E1B32] transition-all text-center text-lg font-mono" />
          <button @click="verifyPassport" 
                  class="w-full py-4 bg-[#9E1B32] text-white rounded-2xl font-black text-lg hover:scale-105 transition-transform shadow-lg">
            Skenēt Pasi 🚀
          </button>
        </div>
        
        <button @click="view = 'admin'" class="mt-12 text-slate-400 text-xs hover:text-slate-600 transition-colors underline">
          Admin paneļis
        </button>
      </div>

      <!-- ADMIN SCREEN -->
      <div v-else-if="view === 'admin'">
        <AdminPanel />
      </div>

      <!-- PASSPORT SCREEN -->
      <div v-else-if="view === 'passport'" class="mt-8">
        <PassportCard :user="user" @go-to-game="view = 'game'" />
      </div>

      <!-- GAME SCREEN -->
      <div v-else-if="view === 'game'" class="mt-8">
        <QuizScreen :gameState="gameState" @submit-answer="submitAnswer" />
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import PassportCard from './components/PassportCard.vue';
import QuizScreen from './components/QuizScreen.vue';
import AdminPanel from './components/AdminPanel.vue';

const API_BASE = import.meta.env.VITE_API_URL;
const user = ref(null);
const passportId = ref('');
const view = ref('passport');
const gameState = ref({ isActive: false, currentQuestion: null });

const verifyPassport = async () => {
  try {
    const res = await fetch(`${API_BASE}/api/passport/verify`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ passportId: passportId.value })
    });
    const data = await res.json();
    if (data.verified) {
      user.value = data.member;
      view.value = 'passport';
    } else {
      alert("Pase netika atrasta! Lūdzu, pārbaudi kodu.");
    }
  } catch (e) {
    alert("Serveris nav sasniedzams!");
  }
};

onMounted(() => {
  setInterval(async () => {
    try {
      const res = await fetch(`${API_BASE}/api/game/state`);
      gameState.value = await res.json();
    } catch (e) {}
  }, 3000);
});

const submitAnswer = (ans) => { alert("Atbilde sūtīta! Gaidi rezultātus."); };
</script>

