<script setup>
import WeatherPreview from './components/WeatherPreview.vue'
import { ref, onMounted, onUnmounted } from 'vue'

const localTime = ref('')
const utcTime = ref('')
let timer = null

function updateTime() {
  const now = new Date()
  localTime.value = now.toLocaleString()
  utcTime.value = now.toISOString().replace('T', ' ').replace(/\..+/, '')
}

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})
onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div>
    <main>
      <WeatherPreview />
      <!----<TheWelcome />!-->
    </main>
    <footer class="footer-info">
      <div class="footer-time">
        <span>本地时间：{{ localTime }}</span> ｜ <span>世界协调时间：{{ utcTime }}</span> ｜ <span>更新时间按当前时区计算</span>
      </div>
      Powered by
      <img src="./assets/logo.svg" alt="Vue Logo" class="footer-logo" />
      Vue &amp;
      <img src="./assets/vite.svg" alt="Vite Logo" class="footer-logo" />
      Vite | Material Design Theme | © 2025
    </footer>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}


@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}

.footer-info {
  width: 100%;
  text-align: center;
  padding: 16px 0 12px 0;
  color: var(--md-on-surface);
  background: transparent;
  font-size: 0.95rem;
  letter-spacing: 0.02em;
  opacity: 0.7;
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 100;
}
.footer-logo {
  height: 22px;
  width: 22px;
  margin: 0 2px;
  vertical-align: middle;
}
.footer-time {
  margin-top: 4px;
  font-size: 0.92em;
  opacity: 0.85;
}
</style>
