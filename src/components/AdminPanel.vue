<template>
  <div class="admin-container p-6 bg-slate-100 min-h-screen font-sans">
    
    <!-- 🔒 LOCK SCREEN -->
    <div v-if="!isAuthenticated" class="flex flex-col items-center justify-center min-h-[80vh] text-center">
      <div class="bg-white p-10 rounded-3xl shadow-2xl border-t-8 border-slate-800 max-w-sm w-full">
        <span class="text-6xl mb-4 block">🔐</span>
        <h2 class="text-2xl font-black text-slate-800 mb-2">Administrators</h2>
        <p class="text-slate-500 mb-6">Lūdzu, ievadi secret kodu, lai piekļūt vadībai.</p>
        
        <input v-model="authPassword" 
               type="password" 
               placeholder="Sēkrētais kods" 
               class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl mb-4 outline-none focus:border-slate-800 transition-all text-center font-mono text-xl" />
        
        <button @click="verifyAdmin" 
                :disabled="isLoading"
                class="w-full py-4 bg-slate-900 text-white rounded-2xl font-bold hover:bg-black transition-all">
          {{ isLoading ? 'Pārbauda...' : 'Atvērt' }}
        </button>
        
        <p v-if="authError" class="text-red-500 text-sm mt-4 font-bold">{{ authError }}</p>
      </div>
    </div>

    <!-- 🎮 ACTUAL ADMIN PANEL (Only shows if authenticated) -->
    <div v-else>
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-3xl font-black text-[#9E1B32]">
          Brikmaņu Kontrolpanelis 🐾
        </h1>
        <button @click="isAuthenticated = false" class="text-xs bg-red-100 text-red-600 px-3 py-1 rounded-full font-bold">
          Iziet
        </button>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <!-- GAME CONTROL -->
        <div class="bg-white p-8 rounded-3xl shadow-xl border-t-8 border-[#9E1B32]">
          <div class="flex items-center gap-3 mb-6">
            <span class="text-3xl">🎮</span>
            <h2 class="text-2xl font-bold text-slate-800">Spēles vadība</h2>
          </div>
          <div class="space-y-6">
            <div>
              <label class="block text-sm font-bold text-slate-500 mb-2">Jautājums</label>
              <input v-model="gameData.question" class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl focus:border-[#9E1B32] outline-none" />
            </div>
            <div>
              <label class="block text-sm font-bold text-slate-500 mb-2">Atbildes varianti (viens katrā rindā)</label>
              <textarea v-model="gameData.options" rows="4" class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl focus:border-[#9E1B32] outline-none"></textarea>
            </div>
            <div>
              <label class="block text-sm font-bold text-slate-500 mb-2">Pareizā atbilde</label>
              <input v-model="gameData.correctAnswer" class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl focus:border-[#9E1B32] outline-none" />
            </div>
            <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border-2 border-slate-200">
              <span class="font-bold text-slate-700">Aktivizēt spēli visiem</span>
              <input type="checkbox" v-model="gameData.isActive" class="w-6 h-6 accent-[#9E1B32]" />
            </div>
            <button @click="updateGame" class="w-full py-4 bg-[#9E1B32] text-white rounded-2xl font-black text-lg hover:bg-red-800 transition-all">
              Sūtīt uz visiem tālruņiem 🚀
            </button>
          </div>
        </div>

        <!-- GUEST PASSES -->
        <div class="bg-white p-8 rounded-3xl shadow-xl border-t-8 border-yellow-500">
          <div class="flex items-center gap-3 mb-6">
            <span class="text-3xl">🎫</span>
            <h2 class="text-2xl font-bold text-slate-800">Izdot viesa pasi</h2>
          </div>
          <div class="space-y-6">
            <div>
              <label class="block text-sm font-bold text-slate-500 mb-2">Viesu vārds</label>
              <input v-model="newGuest.name" class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl focus:border-yellow-500 outline-none" />
            </div>
            <div>
              <label class="block text-sm font-bold text-slate-500 mb-2">Zariņš</label>
              <input v-model="newGuest.branch" class="w-full p-4 bg-slate-50 border-2 border-slate-200 rounded-2xl focus:border-yellow-500 outline-none" />
            </div>
            <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border-2 border-slate-200">
              <span class="font-bold text-slate-700">VIP statuss</span>
              <input type="checkbox" v-model="newGuest.isVIP" class="w-6 h-6 accent-yellow-600" />
            </div>
            <button @click="issuePass" class="w-full py-4 bg-slate-900 text-white rounded-2xl font-black text-lg hover:bg-black transition-all">
              Ģenerēt Pasi ⚡
            </button>
          </div>
          <div v-if="generatedPass" class="mt-8 p-6 bg-yellow-100 border-4 border-dashed border-yellow-500 rounded-3xl text-center animate-bounce">
            <h3 class="text-4xl font-black text-slate-900 mb-2">{{ generatedPass.id }}</h3>
            <p class="text-sm text-yellow-800 font-medium">Pase izveidota!</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';

const API_BASE = import.meta.env.VITE_API_URL;

// AUTH STATE
const isAuthenticated = ref(false);
const authPassword = ref('');
const authError = ref('');
const isLoading = ref(false);

const verifyAdmin = async () => {
  isLoading.value = true;
  authError.value = '';
  try {
    const res = await fetch(`${API_BASE}/api/admin/auth`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: authPassword.value })
    });
    const data = await res.json();
    if (data.authorized) {
      isAuthenticated.value = true;
      authPassword.value = '';
    } else {
      authError.value = data.message || "Kļūdains kods!";
    }
  } catch (e) {
    authError.value = "Kļūda łączamies ar serveri!";
  } finally {
    isLoading.value = false;
  }
};

// GAME & GUEST LOGIC
const gameData = reactive({ question: '', options: '', correctAnswer: '', isActive: false });
const newGuest = reactive({ name: '', branch: '', isVIP: false });
const generatedPass = ref(null);

const updateGame = async () => {
  await fetch(`${API_BASE}/api/game/control`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(gameData)
  });
  alert("Sūtīts!");
};

const issuePass = async () => {
  const res = await fetch(`${API_BASE}/api/passport/create`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newGuest)
  });
  const data = await res.json();
  if (data.member) {
    generatedPass.value = data.member;
    newGuest.name = ''; newGuest.branch = ''; newGuest.isVIP = false;
  }
};
</script>
