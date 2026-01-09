<script setup>
import MountainPicker from './components/MountainPicker.vue'
</script>

<template>
  <MountainPicker />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Comic Sans MS', 'Arial Rounded MT Bold', 'Helvetica Rounded', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(180deg, 
    #87CEEB 0%,      /* 天蓝色 */
    #E0F6FF 40%,     /* 淡蓝色 */
    #FFF9E6 60%,     /* 淡黄色 */
    #90EE90 100%     /* 淡绿色 */
  );
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
}

body::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    radial-gradient(circle at 20% 80%, rgba(255, 182, 193, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(173, 216, 230, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 50% 50%, rgba(255, 255, 224, 0.3) 0%, transparent 50%);
  pointer-events: none;
  z-index: 0;
}

#app {
  padding: 1rem;
  position: relative;
  z-index: 1;
}

/* 添加一些可爱的装饰元素 */
body::after {
  content: '🌈';
  position: fixed;
  top: 10%;
  right: 5%;
  font-size: 4rem;
  opacity: 0.3;
  animation: float 6s infinite ease-in-out;
  pointer-events: none;
  z-index: 0;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-20px) rotate(5deg);
  }
}
</style>
