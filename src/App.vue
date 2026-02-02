<template>
  <ThreeScene />
</template>

<script setup>
  import { defineAsyncComponent } from 'vue'

  const ThreeScene = defineAsyncComponent(() =>
    import('./components/ThreeScene.vue')
  )
</script>