<template>
  <div class="min-h-screen bg-gray-100 font-sans text-slate-900">
    <nav class="bg-[#9E1B32] p-4 text-white flex justify-between items-center shadow-lg">
      <h1 class="text-xl font-bold tracking-wider">Brikmaņu Dzimta 2024</h1>
      <div v-if="user" class="text-sm bg-white text-[#9E1B32] px-3 py-1 rounded-full font-bold">
        Sveiks, {{ user.name }}!
      </div>
    </nav>

    <main class="p-4 max-w-md mx-auto">
      <!-- LOGIN SCREEN -->
      <div v-if="!user && view !== 'admin'" class="mt-20 text-center">
        <h2 class="text-2xl font-bold mb-4">Ienākt sistēmā</h2>
        <input v-model="passportId" placeholder="Ievadi savu pases ID" 
               class="w-full p-4 rounded-xl border-2 border-gray-300 mb-4 outline-none" />
        <button @click="verifyPassport" class="w-full bg-[#9E1B32] text-white p-4 rounded-xl font-bold">
          Skenēt Pasi
        </button>
        <button @click="view = 'admin'" class="mt-8 text-xs text-gray-400 underline">Admin paneļis</button>
      </div>

      <!-- ADMIN SCREEN -->
      <div v-if="view === 'admin'">
        <AdminPanel />
      </div>

      <!-- PASSPORT SCREEN -->
      <div v-else-if="view === 'passport'">
        <PassportCard :user="user" @go-to-game="view = 'game'" />
      </div>

      <!-- GAME SCREEN -->
      <div v-else-if="view === 'game'">
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
      alert("Pase netikla atrasta!");
    }
  } catch (e) {
    alert("Kļūda łączamies ar serveri!");
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

const submitAnswer = (ans) => { console.log("Answer:", ans); };
</script>
