<script setup>
import WeatherPreview from './components/WeatherPreview.vue'
</script>

<template>


  <main>
    <WeatherPreview />
    <!----<TheWelcome />!-->
  </main>
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
</style>
