<template>
  <div class="container-fluid">
    <h4 class="text-center my-2" style="color: aliceblue;" @click="showVideo = true">Conway's Game of Life
    </h4>

    <div class="text-center">
      <button class="btn btn-control my-2" @click="showVideo = true">Play Video</button>
    </div>


    <div v-if="showVideo" class="video-modal-overlay" @click.self="showVideo = false">
      <iframe width="100%" height="35%"
        src="https://www.youtube.com/embed/kic2yqJ_Gs0?si=SVuy8B6QQsICkBQS&amp;controls=0" title="YouTube video player"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    </div>

    <div class="d-flex justify-content-center text-center my-3 flex-wrap">
      <button class="btn btn-control m-1" @click="toggleGame">{{ isRunning ? 'Stop' : 'Start' }}</button>
      <button class="btn btn-control m-1" @click="clearGrid">Clear</button>
      <button class="btn btn-control m-1" @click="randomizeGrid">Randomize</button>
      <select class="btn btn-control m-1" @change="selectPattern($event)">
        <option value="">Select Pattern</option>
        <option v-for="pattern in patterns" :key="pattern.name" :value="pattern.name">{{ pattern.name }}</option>
      </select>
      <!-- <button class="btn btn-control" @click="logLiveCells">Log Live Cells</button>  -->
    </div>
    <div class="grid-container">
      <div class="grid">
        <div v-for="(cell, index) in grid" :key="index" :class="['cell', { alive: cell }]" @click="toggleCell(index)">
        </div>
      </div>
    </div>
    <div class="row justify-content-center text-center my-2">
      <div class="col-6 col-sm-3 mx-3 my-1">Generation: {{ generation }}</div>
      <div class="col-6 col-sm-3 mx-3 my-1">Population: {{ population }}</div>
    </div>
    <div class="row justify-content-center text-center align-items-center my-2">
      <div class="col-12 col-sm-auto my-1">Speed: {{ intervalSpeed }} ms</div>
      <div class="col-12 col-sm-auto d-flex justify-content-center align-items-center my-1">
        <button class="btn btn-control mx-1" @click="decreaseSpeed">&lt;</button>
        <button class="btn btn-control mx-1" @click="increaseSpeed">&gt;</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onUnmounted, computed } from 'vue';
import patterns from './config.json';

export default {
  setup() {
    const rows = 60;
    const cols = 60;
    const grid = ref(Array(rows * cols).fill(false));
    const generation = ref(0);
    const interval = ref(null);
    const intervalSpeed = ref(150); // Default speed
    const isRunning = ref(false);
    const showVideo = ref(false);

    const toggleCell = (index) => {
      grid.value[index] = !grid.value[index];
    };

    const toggleGame = () => {
      if (isRunning.value) {
        stopGame();
      } else {
        startGame();
      }
    };

    const startGame = () => {
      if (interval.value) return;
      interval.value = setInterval(() => {
        nextGeneration();
        generation.value += 1;
      }, intervalSpeed.value);
      isRunning.value = true;
    };

    const stopGame = () => {
      clearInterval(interval.value);
      interval.value = null;
      isRunning.value = false;
    };

    const clearGrid = () => {
      grid.value = Array(rows * cols).fill(false);
      generation.value = 0;
    };

    const randomizeGrid = () => {
      grid.value = grid.value.map(() => Math.random() < 0.3); // 30% chance to be alive
    };

    const nextGeneration = () => {
      const newGrid = [...grid.value];
      for (let i = 0; i < rows * cols; i++) {
        const x = i % cols;
        const y = Math.floor(i / cols);
        const neighbors = [
          [x - 1, y - 1], [x, y - 1], [x + 1, y - 1],
          [x - 1, y], [x + 1, y],
          [x - 1, y + 1], [x, y + 1], [x + 1, y + 1]
        ].filter(([nx, ny]) => nx >= 0 && ny >= 0 && nx < cols && ny < rows);

        const aliveNeighbors = neighbors.filter(([nx, ny]) => grid.value[ny * cols + nx]).length;

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

    const increaseSpeed = () => {
      if (intervalSpeed.value > 50) {
        intervalSpeed.value -= 50;
        updateSpeed();
      }
    };

    const decreaseSpeed = () => {
      if (intervalSpeed.value < 1000) {
        intervalSpeed.value += 50;
        updateSpeed();
      }
    };

    const population = computed(() => grid.value.filter(cell => cell).length);

    onUnmounted(() => {
      stopGame();
    });

    const selectPattern = (event) => {
      const selectedPattern = patterns.find(pattern => pattern.name === event.target.value);
      if (selectedPattern) {
        clearGrid();
        selectedPattern.pattern.forEach(([x, y]) => {
          grid.value[y * cols + x] = true;
        });
      }
    };

    const logLiveCells = () => {
      const liveCells = [];
      for (let i = 0; i < rows * cols; i++) {
        if (grid.value[i]) {
          const x = i % cols;
          const y = Math.floor(i / cols);
          liveCells.push([x, y]);
        }
      }
      console.log(liveCells);
    };

    return {
      grid,
      generation,
      population,
      intervalSpeed,
      toggleCell,
      toggleGame,
      startGame,
      stopGame,
      clearGrid,
      randomizeGrid,
      increaseSpeed,
      decreaseSpeed,
      isRunning,
      selectPattern,
      patterns,
      logLiveCells,
      showVideo,
    };
  }
};
</script>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

.container-fluid,
.info,
.grid,
.cell {
  font-family: 'Press Start 2P', cursive;
}

.info {
  color: #fff;
}

.grid-container {
  display: flex;
  justify-content: center;
}

.grid {
  display: grid;
  grid-template-columns: repeat(60, minmax(5px, 10px));
  gap: 0;
}

.cell {
  width: 10px;
  height: 10px;
  background-color: transparent;
  box-shadow: 0 0 0.2px 0.2px white;
}

.cell.alive {
  background-color: white;
}

.btn-control {
  background-color: transparent;
  border: 1px solid rgba(255, 255, 255, 0.5);
  color: rgba(255, 255, 255, 0.8);
  margin: 0 5px;
  padding: 5px 10px;
  font-size: 12px;
  transition: background-color 0.3s, color 0.3s;
}

.btn-control:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.btn.btn-control option {
  background-color: #feb47b;
  color: white;
}

@media (max-width: 768px) {
  .grid {
    grid-template-columns: repeat(60, 8px);
  }

  .cell {
    width: 8px;
    height: 8px;
  }

  .btn-control {
    font-size: 10px;
    padding: 3px 8px;
  }

  .container-fluid h4 {
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .grid {
    grid-template-columns: repeat(60, 5px);
  }

  .cell {
    width: 5px;
    height: 5px;
  }

  .btn-control {
    font-size: 8px;
    padding: 2px 6px;
  }

  .container-fluid h4 {
    font-size: 14px;
  }

  .grid-container {
    overflow-x: auto;
  }
}

.video-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
</style>
