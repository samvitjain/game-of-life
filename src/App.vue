<!-- src/App.vue -->
<template>
  <div id="app">
    <GameOfLife />
  </div>
</template>

<script>
import GameOfLife from './components/GameOfLife.vue';

export default {
  name: 'App',
  components: {
    GameOfLife
  }
}
</script>

<style>
/* Add any global styles here */
</style>
