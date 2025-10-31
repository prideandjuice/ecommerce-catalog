<script setup>
import ProductViewer from './components/ProductViewer.vue'
</script>

<template>
  <ProductViewer />
</template>

<style>
/* Reset body dan html untuk full screen background */
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
}

#app {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
}
</style>
