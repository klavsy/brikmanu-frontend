<!-- frontend/src/components/QuizScreen.vue -->
<template>
  <div class="quiz-container p-4">
    <!-- Question Area -->
    <div class="question-card bg-white p-6 rounded-3xl shadow-lg mb-6">
      <h2 class="text-xl font-bold text-center mb-6">{{ gameState.currentQuestion.question }}</h2>
      
      <div class="grid grid-cols-1 gap-3">
        <button v-for="opt in gameState.currentQuestion.options" 
                @click="submitAnswer(opt)"
                class="p-4 border-2 border-gray-200 rounded-2xl hover:border-[#9E1B32] transition font-medium">
          {{ opt }}
        </button>
      </div>
    </div>

    <!-- THE HELP BUTTON -->
    <button @click="askPrincessForHelp" 
            class="w-full py-4 bg-gradient-to-r from-yellow-400 to-yellow-600 text-slate-900 rounded-2xl font-black flex items-center justify-center gap-2 shadow-lg hover:scale-105 transition-transform">
      <span>🐾</span> Lūgums Princesei (Hint)
    </button>

    <!-- THE SIRI-LIKE FLOATING ORB -->
    <PrincessSiri :isActive="isPrincessActive" :message="princessMessage" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import PrincessSiri from './PrincessSiri.vue';

const props = defineProps(['gameState']);
const isPrincessActive = ref(false);
const princessMessage = ref('');

const askPrincessForHelp = async () => {
  isPrincessActive.value = true;
  princessMessage.value = "Princese domā... *murr*";

  try {
    const res = await fetch('https://your-render-url.onrender.com/api/princess-hint', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ 
        question: props.gameState.currentQuestion.question, 
        answer: props.gameState.currentQuestion.correctAnswer 
      })
    });
    const data = await res.json();
    princessMessage.value = data.hint;
  } catch (e) {
    princessMessage.value = "Kaut kas novirzās manās spiļās... mēģini vēlreiz!";
  }

  // Automatically hide the orb after 8 seconds
  setTimeout(() => {
    isPrincessActive.value = false;
    princessMessage.value = '';
  }, 8000);
};
</script>
