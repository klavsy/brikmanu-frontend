<template>
  <div class="p-4 max-w-4xl mx-auto font-sans">
    
    <!-- LOCK SCREEN -->
    <div v-if="!isAuthenticated" class="flex flex-col items-center justify-center min-h-[70vh]">
      <div class="bg-white p-8 rounded-3xl shadow-xl border-b-4 border-slate-800 w-full max-w-xs text-center">
        <div class="text-5xl mb-4">🔐</div>
        <h2 class="text-2xl font-black text-slate-800 mb-4">Admin Piekļuve</h2>
        <input v-model="authPassword" type="password" placeholder="Sēkrētais kods" 
               class="w-full p-4 bg-slate-100 border-2 border-slate-200 rounded-2xl mb-4 outline-none focus:border-slate-800 text-center font-mono" />
        <button @click="verifyAdmin" class="w-full py-4 bg-slate-900 text-white rounded-2xl font-bold hover:bg-black transition-all">
          Atslēgt
        </button>
        <p v-if="authError" class="text-red-500 text-xs mt-3 font-bold">{{ authError }}</p>
      </div>
    </div>

    <!-- DASHBOARD -->
    <div v-else class="space-y-8">
      <div class="flex justify-between items-center">
        <h1 class="text-2xl font-black text-slate-800">Kontrolpanelis 🐾</h1>
        <button @click="isAuthenticated = false" class="px-4 py-2 bg-red-100 text-red-600 rounded-full text-xs font-bold">Iziet</button>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        
        <!-- GAME MASTER CARD -->
        <div class="bg-white p-6 rounded-3xl shadow-lg border-l-8 border-[#9E1B32]">
          <div class="flex items-center gap-2 mb-6">
            <span class="text-2xl">🎮</span>
            <h2 class="text-xl font-bold text-slate-800">Sagatavot Jautājumu</h2>
          </div>
          
          <div class="space-y-4">
            <div class="group">
              <label class="text-[10px] uppercase font-black text-slate-400 ml-2">Jautājums</label>
              <input v-model="gameData.question" class="w-full p-3 bg-slate-50 border-2 border-slate-100 rounded-xl outline-none focus:border-[#9E1B32] transition-all" />
            </div>
            <div class="group">
              <label class="text-[10px] uppercase font-black text-slate-400 ml-2">Svariants (viens katrā rindā)</label>
              <textarea v-model="gameData.options" rows="3" class="w-full p-3 bg-slate-50 border-2 border-slate-100 rounded-xl outline-none focus:border-[#9E1B32] transition-all"></textarea>
            </div>
            <div class="group">
              <label class="text-[10px] uppercase font-black text-slate-400 ml-2">Pareizā atbilde</label>
              <input v-model="gameData.correctAnswer" class="w-full p-3 bg-slate-50 border-2 border-slate-100 rounded-xl outline-none focus:border-[#9E1B32] transition-all" />
            </div>
            
            <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border-2 border-slate-100">
              <span class="font-bold text-slate-600 text-sm">Sūtīt visiem tālruņiem?</span>
              <input type="checkbox" v-model="gameData.isActive" class="w-5 h-5 accent-[#9E1B32]" />
            </div>

            <button @click="updateGame" class="w-full py-4 bg-[#9E1B32] text-white rounded-2xl font-black hover:bg-red-800 shadow-lg transition-all active:scale-95">
              SŪTĪT 🚀
            </button>
          </div>
        </div>

        <!-- PASS ISSUANCE CARD -->
        <div class="bg-white p-6 rounded-3xl shadow-lg border-l-8 border-yellow-500">
          <div class="flex items-center gap-2 mb-6">
            <span class="text-2xl">🎫</span>
            <h2 class="text-xl font-bold text-slate-800">Izdot Viesu Pasi</h2>
          </div>

          <div class="space-y-4">
            <div class="group">
              <label class="text-[10px] uppercase font-black text-slate-400 ml-2">Vārds</label>
              <input v-model="newGuest.name" class="w-full p-3 bg-slate-50 border-2 border-slate-100 rounded-xl outline-none focus:border-yellow-500 transition-all" />
            </div>
            <div class="group">
              <label class="text-[10px] uppercase font-black text-slate-400 ml-2">Zariņš / Grupa</label>
              <input v-model="newGuest.branch" class="w-full p-3 bg-slate-50 border-2 border-slate-100 rounded-xl outline-none focus:border-yellow-500 transition-all" />
            </div>
            <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border-2 border-slate-100">
              <span class="font-bold text-slate-600 text-sm">VIP Statuss</span>
              <input type="checkbox" v-model="newGuest.isVIP" class="w-5 h-5 accent-yellow-600" />
            </div>
            <button @click="issuePass" class="w-full py-4 bg-slate-900 text-white rounded-2xl font-black hover:bg-black shadow-lg transition-all active:scale-95">
              ĢENERĒT P ASI ⚡
            </button>
          </div>

          <div v-if="generatedPass" class="mt-6 p-6 bg-yellow-50 border-2 border-dashed border-yellow-400 rounded-3xl text-center animate-pulse">
            <p class="text-[10px] font-black text-yellow-600 uppercase mb-1">Kods izveidots!</p>
            <h3 class="text-4xl font-black text-slate-900">{{ generatedPass.id }}</h3>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
const API_BASE = import.meta.env.VITE_API_URL;

const isAuthenticated = ref(false);
const authPassword = ref('');
const authError = ref('');

const gameData = reactive({ question: '', options: '', correctAnswer: '', isActive: false });
const newGuest = reactive({ name: '', branch: '', isVIP: false });
const generatedPass = ref(null);

const verifyAdmin = async () => {
  try {
    const res = await fetch(`${API_BASE}/api/admin/auth`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: authPassword.value })
    });
    if (res.ok) {
      isAuthenticated.value = true;
    } else {
      authError.value = "Sēkrētais kods ir kļūdains!";
    }
  } catch (e) { authError.value = "Kļūda łączamies ar serveri!"; }
};

const updateGame = async () => {
  try {
    await fetch(`${API_BASE}/api/game/control`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(gameData)
    });
    alert("Jautājums sūtīts visiem!");
  } catch (e) { alert("Sūtīšanas kļūda!"); }
};

const issuePass = async () => {
  try {
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
  } catch (e) { alert("Kļūda ģenerējot pasi!"); }
};
</script>

