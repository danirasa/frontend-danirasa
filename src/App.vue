<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
import SensorDashboard from './components/SensorDashboard.vue'
</script>

<template>
  <main>
    <SensorDashboard />
  </main>
</template>

<style scoped>
body {
  background-color: #f1f5f9;
  margin: 0;
  padding: 0;
}
</style>
