<!-- frontend/src/components/PrincessSiri.vue -->
<template>
  <div class="siri-container" v-if="isActive">
    <!-- The Glowing Orb -->
    <div class="orb-wrapper">
      <div class="blob blob-1"></div>
      <div class="blob blob-2"></div>
      <div class="blob blob-3"></div>
      <div class="cat-face">🐾</div>
    </div>

    <!-- The Response Text -->
    <div class="response-bubble" v-if="message">
      <p class="purr-text"><i>*murr*</i></p>
      <p class="message-text">{{ message }}</p>
    </div>
  </div>
</template>

<script setup>
defineProps(['isActive', 'message']);
</script>

<style scoped>
.siri-container {
  position: fixed;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 9999;
  transition: all 0.5s ease-in-out;
}

/* The "Siri" Orb Animation */
.orb-wrapper {
  position: relative;
  width: 100px;
  height: 100px;
  filter: blur(8px);
}

.blob {
  position: absolute;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  opacity: 0.8;
  animation: move 4s infinite alternate ease-in-out;
}

.blob-1 {
  background: #9E1B32; /* Latvian Red */
  top: 10px; left: 10px;
  animation-delay: 0s;
}

.blob-2 {
  background: #D4AF37; /* Princess Gold */
  top: 20px; right: 10px;
  animation-delay: -1s;
}

.blob-3 {
  background: #FFFFFF; /* Pure White */
  bottom: 10px; left: 30px;
  animation-delay: -2s;
}

.cat-face {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 2.5rem;
  z-index: 10;
  filter: blur(0px); /* Keep the paw sharp */
}

@keyframes move {
  from { transform: translate(0, 0) scale(1); }
  to { transform: translate(20px, 10px) scale(1.2); }
}

/* The Floating Text Bubble */
.response-bubble {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border: 2px solid #D4AF37;
  padding: 15px 20px;
  border-radius: 25px 25px 25px 5px;
  margin-top: 20px;
  max-width: 300px;
  box-shadow: 0 15px 35px rgba(0,0,0,0.2);
  animation: slideUp 0.4s ease-out;
}

.purr-text {
  color: #9E1B32;
  font-size: 0.75rem;
  font-weight: bold;
}

.message-text {
  color: #333;
  font-size: 1rem;
  line-height: 1.4;
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
</style>
