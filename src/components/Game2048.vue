<script setup>
import { ref, onMounted, onUnmounted, computed, reactive } from 'vue';
// Import VantaJS fog effect
import FOG from 'vanta/dist/vanta.fog.min';
import * as THREE from 'three';

// Vanta.js background
const vantaEffect = ref(null);
const vantaContainer = ref(null);

// Game state
const board = ref([]);
const score = ref(0);
const gameOver = ref(false);
const won = ref(false);

// Touch handling
const touchStartX = ref(0);
const touchStartY = ref(0);
const touchEndX = ref(0);
const touchEndY = ref(0);

// Initialize the game board (4x4 grid)
const initBoard = () => {
  board.value = Array(4).fill().map(() => Array(4).fill(0));
  addRandomTile();
  addRandomTile();
  score.value = 0;
  gameOver.value = false;
  won.value = false;
};

// Add a random tile (2 or 4) to an empty cell
const addRandomTile = () => {
  const emptyCells = [];

  // Find all empty cells
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 4; j++) {
      if (board.value[i][j] === 0) {
        emptyCells.push({ row: i, col: j });
      }
    }
  }

  if (emptyCells.length > 0) {
    // Randomly select an empty cell
    const randomCell = emptyCells[Math.floor(Math.random() * emptyCells.length)];
    // 90% chance for a 2, 10% chance for a 4
    board.value[randomCell.row][randomCell.col] = Math.random() < 0.9 ? 2 : 4;
  }
};

// Check if the game is over
const checkGameOver = () => {
  // Check if there are any empty cells
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 4; j++) {
      if (board.value[i][j] === 0) {
        return false;
      }
    }
  }

  // Check if there are any possible merges horizontally
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 3; j++) {
      if (board.value[i][j] === board.value[i][j + 1]) {
        return false;
      }
    }
  }

  // Check if there are any possible merges vertically
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 4; j++) {
      if (board.value[i][j] === board.value[i + 1][j]) {
        return false;
      }
    }
  }

  return true;
};

// Move tiles in a direction (left, right, up, down)
const moveTiles = (direction) => {
  if (gameOver.value) return;

  const originalBoard = JSON.stringify(board.value);
  let moved = false;

  // Clone the board to work with
  const newBoard = board.value.map(row => [...row]);

  // Process the board based on direction
  if (direction === 'left') {
    for (let i = 0; i < 4; i++) {
      const row = newBoard[i].filter(val => val !== 0);
      for (let j = 0; j < row.length - 1; j++) {
        if (row[j] === row[j + 1]) {
          row[j] *= 2;
          row[j + 1] = 0;
          score.value += row[j];
          if (row[j] === 2048) won.value = true;
        }
      }
      const filteredRow = row.filter(val => val !== 0);
      while (filteredRow.length < 4) filteredRow.push(0);
      newBoard[i] = filteredRow;
    }
  } else if (direction === 'right') {
    for (let i = 0; i < 4; i++) {
      const row = newBoard[i].filter(val => val !== 0);
      for (let j = row.length - 1; j > 0; j--) {
        if (row[j] === row[j - 1]) {
          row[j] *= 2;
          row[j - 1] = 0;
          score.value += row[j];
          if (row[j] === 2048) won.value = true;
        }
      }
      const filteredRow = row.filter(val => val !== 0);
      while (filteredRow.length < 4) filteredRow.unshift(0);
      newBoard[i] = filteredRow;
    }
  } else if (direction === 'up') {
    for (let j = 0; j < 4; j++) {
      let col = [];
      for (let i = 0; i < 4; i++) {
        if (newBoard[i][j] !== 0) col.push(newBoard[i][j]);
      }
      for (let i = 0; i < col.length - 1; i++) {
        if (col[i] === col[i + 1]) {
          col[i] *= 2;
          col[i + 1] = 0;
          score.value += col[i];
          if (col[i] === 2048) won.value = true;
        }
      }
      col = col.filter(val => val !== 0);
      while (col.length < 4) col.push(0);
      for (let i = 0; i < 4; i++) {
        newBoard[i][j] = col[i];
      }
    }
  } else if (direction === 'down') {
    for (let j = 0; j < 4; j++) {
      let col = [];
      for (let i = 0; i < 4; i++) {
        if (newBoard[i][j] !== 0) col.push(newBoard[i][j]);
      }
      for (let i = col.length - 1; i > 0; i--) {
        if (col[i] === col[i - 1]) {
          col[i] *= 2;
          col[i - 1] = 0;
          score.value += col[i];
          if (col[i] === 2048) won.value = true;
        }
      }
      col = col.filter(val => val !== 0);
      while (col.length < 4) col.unshift(0);
      for (let i = 0; i < 4; i++) {
        newBoard[i][j] = col[i];
      }
    }
  }

  // Check if the board has changed
  if (JSON.stringify(newBoard) !== originalBoard) {
    moved = true;
    board.value = newBoard;
    addRandomTile();
  }

  // Check if game is over
  if (checkGameOver()) {
    gameOver.value = true;
  }

  return moved;
};

// Handle keyboard events
const handleKeyDown = (event) => {
  switch (event.key) {
    case 'ArrowLeft':
      moveTiles('left');
      break;
    case 'ArrowRight':
      moveTiles('right');
      break;
    case 'ArrowUp':
      moveTiles('up');
      break;
    case 'ArrowDown':
      moveTiles('down');
      break;
  }
};

// Handle touch events
const handleTouchStart = (event) => {
  touchStartX.value = event.touches[0].clientX;
  touchStartY.value = event.touches[0].clientY;
  // Prevent default to avoid scrolling when starting touch inside game board
  event.preventDefault();
};

const handleTouchMove = (event) => {
  // Prevent scrolling when swiping inside the game board
  event.preventDefault();
};

const handleTouchEnd = (event) => {
  touchEndX.value = event.changedTouches[0].clientX;
  touchEndY.value = event.changedTouches[0].clientY;

  const deltaX = touchEndX.value - touchStartX.value;
  const deltaY = touchEndY.value - touchStartY.value;

  // Determine swipe direction based on which delta is larger
  if (Math.abs(deltaX) > Math.abs(deltaY)) {
    // Horizontal swipe
    if (deltaX > 20) {
      moveTiles('right');
    } else if (deltaX < -20) {
      moveTiles('left');
    }
  } else {
    // Vertical swipe
    if (deltaY > 20) {
      moveTiles('down');
    } else if (deltaY < -20) {
      moveTiles('up');
    }
  }
};

// Get tile color based on value
const getTileColor = (value) => {
  const colors = {
    0: '#eee4da59',
    2: '#FFF5E0',
    4: '#FFE4C9',
    8: '#FFCDA8',
    16: '#FFB086',
    32: '#FF9364',
    64: '#FF7F50',
    128: '#F9D5E3',
    256: '#F2B5D4',
    512: '#E895C2',
    1024: '#E87CBC',
    2048: '#E665B1'
  };
  return colors[value] || '#E665B1';
};

// Get text color based on tile value
const getTileTextColor = (value) => {
  return value <= 4 ? '#776e65' : '#f9f6f2';
};

// Get font size based on tile value
const getTileFontSize = (value) => {
  if (value < 10) return '36px';
  if (value < 100) return '32px';
  if (value < 1000) return '24px';
  return '20px';
};

// Best score tracking
const bestScore = ref(localStorage.getItem('bestScore') ? parseInt(localStorage.getItem('bestScore')) : 0);

// Update best score when current score changes
const updateBestScore = () => {
  if (score.value > bestScore.value) {
    bestScore.value = score.value;
    localStorage.setItem('bestScore', bestScore.value.toString());
  }
};

// Check game state when the board changes
const checkGameState = () => {
  // Check for win condition
  if (board.value.some(row => row.some(cell => cell === 2048))) {
    won.value = true;
  }

  // Check for game over
  if (checkGameOver()) {
    gameOver.value = true;
  }
};

// Initialize the game when component is mounted
onMounted(() => {
  initBoard();
  window.addEventListener('keydown', handleKeyDown);

  // Initialize VantaJS fog effect
  if (!vantaEffect.value) {
    vantaEffect.value = FOG({
      el: vantaContainer.value,
      THREE,
      mouseControls: true,
      touchControls: true,
      gyroControls: false,
      minHeight: 200.00,
      minWidth: 200.00,
      highlightColor: 0xF9D5E3, // Light pink similar to the 128 tile
      midtoneColor: 0xFFE4C9,   // Light peach similar to the 4 tile
      lowlightColor: 0xFF9364,  // Orange similar to the 32 tile
      baseColor: 0xFFF5E0,      // Light cream similar to the 2 tile
      blurFactor: 0.60,
      speed: 1.50,
      zoom: 0.70
    });
  }
});

// Clean up event listeners when component is unmounted
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);

  // Clean up VantaJS effect
  if (vantaEffect.value) {
    vantaEffect.value.destroy();
  }
});
</script>

<template>
  <div class="vanta-background" ref="vantaContainer">
    <div class="game-container">
      <div class="header">
        <h1>2048</h1>
        <div class="scores">
          <div class="score-box">
            <div class="score-label">Score</div>
            <div class="score-value">{{ score }}</div>
          </div>
        </div>
      </div>

      <div class="controls">
        <button @click="initBoard" class="new-game-btn">New Game</button>
      </div>

      <div
        class="game-board"
        @touchstart="handleTouchStart"
        @touchmove="handleTouchMove"
        @touchend="handleTouchEnd"
      >
        <div v-for="(row, rowIndex) in board" :key="`row-${rowIndex}`" class="row">
          <div
            v-for="(cell, colIndex) in row"
            :key="`cell-${rowIndex}-${colIndex}`"
            class="cell"
            :style="{
              backgroundColor: getTileColor(cell),
              color: getTileTextColor(cell),
              fontSize: getTileFontSize(cell)
            }"
          >
            {{ cell !== 0 ? cell : '' }}
          </div>
        </div>
      </div>

      <div v-if="gameOver" class="game-message game-over">
        <div class="message">Game Over!</div>
        <button @click="initBoard" class="restart-btn">Try Again</button>
      </div>

      <div v-if="won && !gameOver" class="game-message game-won">
        <div class="message">You Win!</div>
        <button @click="initBoard" class="restart-btn">Play Again</button>
      </div>

      <div class="instructions">
        <p>Use arrow keys to move the tiles. When two tiles with the same number touch, they merge into one!</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.vanta-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: -1;
  overflow: hidden;
  background-color: #FFF5E0; /* Match the base color of VantaJS */
}

.game-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background-color: rgba(255, 255, 255, 0.85);
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  margin-top: 30px;
  margin-bottom: 30px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

h1 {
  font-size: 48px;
  font-weight: bold;
  color: #776e65;
  margin: 0;
}

.scores {
  display: flex;
  gap: 10px;
}

.score-box {
  background-color: #bbada0;
  padding: 10px 15px;
  border-radius: 6px;
  color: white;
  text-align: center;
  min-width: 100px;
}

.score-label {
  font-size: 14px;
  text-transform: uppercase;
}

.score-value {
  font-size: 20px;
  font-weight: bold;
}

.controls {
  margin-bottom: 20px;
}

.new-game-btn {
  background-color: #8f7a66;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s;
}

.new-game-btn:hover {
  background-color: #9f8b77;
}

.game-board {
  background-color: #bbada0;
  border-radius: 6px;
  padding: 15px;
  position: relative;
  width: calc(100% - 30px); /* Account for padding */
  aspect-ratio: 1 / 1; /* Keep the board square */
  max-width: 450px; /* Match roughly with the cell sizes */
  margin: 0 auto;
}

.row {
  display: flex;
  margin-bottom: 15px;
  height: calc(25% - 11.25px); /* 25% height minus part of the gap */
}

.row:last-child {
  margin-bottom: 0;
}

.cell {
  width: calc(25% - 11.25px); /* 25% width minus part of the gap */
  height: 100%;
  background-color: #eee4da59;
  border-radius: 6px;
  margin-right: 15px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 36px;
  font-weight: bold;
  transition: all 0.1s ease-in-out;
}

.cell:last-child {
  margin-right: 0;
}

.game-message {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(238, 228, 218, 0.73);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 6px;
  z-index: 10;
}

.message {
  font-size: 48px;
  font-weight: bold;
  color: #776e65;
  margin-bottom: 20px;
}

.restart-btn {
  background-color: #8f7a66;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

.instructions {
  margin-top: 20px;
  color: #776e65;
  text-align: center;
}

/* Responsive adjustments */
@media (max-width: 500px) {
  .game-board {
    padding: 10px;
    width: calc(100% - 20px); /* Account for the smaller padding */
  }

  .row {
    margin-bottom: 10px;
    height: calc(25% - 7.5px); /* Adjust for smaller margins */
  }

  .cell {
    width: calc(25% - 7.5px); /* Adjust for smaller margins */
    margin-right: 10px;
    font-size: 24px;
  }

  h1 {
    font-size: 36px;
  }

  .score-box {
    min-width: 80px;
    padding: 8px 12px;
  }

  .score-value {
    font-size: 18px;
  }
}
</style>

