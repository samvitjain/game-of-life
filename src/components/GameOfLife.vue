<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Conway's Game of Life</h1>
    <div class="text-center mb-3">
      <button class="btn btn-primary" @click="startGame">Start</button>
      <button class="btn btn-secondary" @click="stopGame">Stop</button>
      <button class="btn btn-danger" @click="clearGrid">Clear</button>
      <button class="btn btn-info" @click="randomizeGrid">Randomize</button>
    </div>
    <div class="info mb-3">
      <p>Current Generation: {{ generation }}</p>
      <p>Population: {{ population }}</p>
      <p>Speed: {{ intervalSpeed }} ms</p>
      <input 
        type="range" 
        min="50" 
        max="1000" 
        step="50" 
        v-model="intervalSpeed" 
        @input="updateSpeed"
      />
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
    const intervalSpeed = ref(150); // Default speed

    const toggleCell = (index) => {
      grid.value[index] = !grid.value[index];
    };

    const startGame = () => {
      if (interval.value) return;
      interval.value = setInterval(() => {
        nextGeneration();
        generation.value += 1;
      }, intervalSpeed.value);
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
      grid.value = grid.value.map(() => Math.random() < 0.3); // 30% chance to be alive
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

    const updateSpeed = () => {
      if (interval.value) {
        stopGame();
        startGame();
      }
    };

    const population = computed(() => grid.value.filter(cell => cell).length);

    onUnmounted(() => {
      stopGame();
    });

    return {
      grid,
      generation,
      population,
      intervalSpeed,
      toggleCell,
      startGame,
      stopGame,
      clearGrid,
      randomizeGrid,
      updateSpeed
    };
  }
};
</script>

<style scoped>

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
  box-shadow: 0 0 0.2px 0.2px white; 
}

.cell.alive {
  background-color: white; /* Alive cells are white */
}
</style>
