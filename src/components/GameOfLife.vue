<template>
  <div class="container mt-4">
    <div class="text-center mb-3">
      <button class="btn btn-primary" @click="startGame">Start</button>
      <button class="btn btn-secondary" @click="stopGame">Stop</button>
      <button class="btn btn-danger" @click="clearGrid">Clear</button>
      <button class="btn btn-info" @click="randomizeGrid">Randomize</button>
    </div>
    <div class="info mb-3">
      <p>Current Generation: {{ generation }}</p>
      <p>Population: {{ population }}</p>
    </div>
    <div class="grid">
      <div
        v-for="(cell, index) in grid"
        :key="index"
        :class="['cell', { alive: cell }]"
        @click="toggleCell(index)"
      ></div>
    </div>
  </div>
</template>

<script>
import { ref, onUnmounted, computed } from 'vue';

export default {
  setup() {
    const grid = ref(Array(900).fill(false)); // 30x30 grid
    const generation = ref(0);
    const interval = ref(null);

    const toggleCell = (index) => {
      grid.value[index] = !grid.value[index];
    };

    const startGame = () => {
      if (interval.value) return;
      interval.value = setInterval(() => {
        nextGeneration();
        generation.value += 1;
      }, 100);
    };

    const stopGame = () => {
      clearInterval(interval.value);
      interval.value = null;
    };

    const clearGrid = () => {
      grid.value = Array(900).fill(false);
      generation.value = 0;
    };

    const randomizeGrid = () => {
      grid.value = grid.value.map(() => Math.random() < 0.3); // 20% chance to be alive
    };

    const nextGeneration = () => {
      const newGrid = [...grid.value];
      for (let i = 0; i < 900; i++) {
        const x = i % 30;
        const y = Math.floor(i / 30);
        const neighbors = [
          [x-1, y-1], [x, y-1], [x+1, y-1],
          [x-1, y],           [x+1, y],
          [x-1, y+1], [x, y+1], [x+1, y+1]
        ].filter(([nx, ny]) => nx >= 0 && ny >= 0 && nx < 30 && ny < 30);
        
        const aliveNeighbors = neighbors.filter(([nx, ny]) => grid.value[ny * 30 + nx]).length;
        
        if (grid.value[i] && (aliveNeighbors < 2 || aliveNeighbors > 3)) {
          newGrid[i] = false;
        }
        if (!grid.value[i] && aliveNeighbors === 3) {
          newGrid[i] = true;
        }
      }
      grid.value = newGrid;
    };

    const population = computed(() => grid.value.filter(cell => cell).length);

    onUnmounted(() => {
      stopGame();
    });

    return {
      grid,
      generation,
      population,
      toggleCell,
      startGame,
      stopGame,
      clearGrid,
      randomizeGrid
    };
  }
};
</script>

<style scoped>
.container {
  background: linear-gradient(to bottom right, #ff7e5f, #feb47b);
  min-height: 100vh;
}

.info {
  color: #fff;
}

.grid {
  display: grid;
  grid-template-columns: repeat(30, 20px);
  gap: 0; /* Remove spacing between cells */
}

.cell {
  width: 20px;
  height: 20px;
  background-color: transparent; /* Dead cells are transparent */
  border: 1px solid #ddd;
}

.cell.alive {
  background-color: white; /* Alive cells are white */
}
</style>
