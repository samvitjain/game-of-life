<!-- src/App.vue -->
<template>
  <div id="gol">
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
#app {
  background: linear-gradient(to bottom right, #ff7e5f, #feb47b);
}
</style>
