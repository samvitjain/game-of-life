<!-- src/App.vue -->
<template>
  <div class="m-2" id="gol">
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
/* html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
} */
#app {
  width: 100%;
  background: linear-gradient(to bottom right, #ff7e5f, #feb47b);
}
</style>
