<template>
  <div class="snake-game-container mx-auto mt-10 max-w-lg p-4">
    <h1 class="text-3xl font-extrabold text-center mb-6 text-blue-600">Game Snake</h1>
    <canvas ref="gameCanvas" class="border border-gray-500 bg-black"></canvas>
    <div v-if="!gameCompleted" class="text-center mt-4">
      <p class="text-lg font-semibold text-gray-700">Score: {{ score }}</p>
      <p class="text-lg font-semibold text-gray-700">Direction: {{ direction }}</p>
    </div>
    <div v-else class="text-center mt-4">
      <p class="text-2xl font-semibold text-gray-800">Game Over! Your score: {{ score }}</p>
      <button @click="restartGame" class="mt-4 bg-blue-500 text-white py-2 px-4 rounded-lg hover:bg-blue-400">Play Again</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvasWidth: 400,
      canvasHeight: 400,
      gridSize: 20,
      snake: [{ x: 160, y: 160 }],
      food: { x: 100, y: 100 },
      direction: 'RIGHT',
      nextDirection: 'RIGHT',
      gameInterval: null,
      score: 0,
      gameCompleted: false,
      intervalTime: 200, // Waktu interval awal
      difficultyIncreaseThreshold: 5 // Skor pada mana kesulitan meningkat
    }
  },
  mounted() {
    this.initGame()
    window.addEventListener('keydown', this.changeDirection)
  },
  beforeUnmount() {
    clearInterval(this.gameInterval)
    window.removeEventListener('keydown', this.changeDirection)
  },
  methods: {
    initGame() {
      const canvas = this.$refs.gameCanvas
      const ctx = canvas.getContext('2d')
      canvas.width = this.canvasWidth
      canvas.height = this.canvasHeight

      this.gameInterval = setInterval(() => {
        this.updateGame()
        this.drawGame(ctx)
        this.checkDifficulty()
      }, this.intervalTime)
    },
    drawGame(ctx) {
      ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)

      // Draw snake
      ctx.fillStyle = '#2d6a4f'
      this.snake.forEach(segment => ctx.fillRect(segment.x, segment.y, this.gridSize, this.gridSize))

      // Draw food
      ctx.fillStyle = '#e63946'
      ctx.fillRect(this.food.x, this.food.y, this.gridSize, this.gridSize)
    },
    updateGame() {
      const head = { ...this.snake[0] }
      this.direction = this.nextDirection

      if (this.direction === 'RIGHT') head.x += this.gridSize
      if (this.direction === 'LEFT') head.x -= this.gridSize
      if (this.direction === 'UP') head.y -= this.gridSize
      if (this.direction === 'DOWN') head.y += this.gridSize

      // Check for collision with walls
      if (head.x < 0 || head.x >= this.canvasWidth || head.y < 0 || head.y >= this.canvasHeight) {
        this.endGame()
        return
      }

      // Check for collision with self
      if (this.snake.some(segment => segment.x === head.x && segment.y === head.y)) {
        this.endGame()
        return
      }

      this.snake.unshift(head)

      // Check if snake has eaten food
      if (head.x === this.food.x && head.y === this.food.y) {
        this.score++
        this.placeFood()
      } else {
        this.snake.pop()
      }
    },
    placeFood() {
      this.food = {
        x: Math.floor(Math.random() * (this.canvasWidth / this.gridSize)) * this.gridSize,
        y: Math.floor(Math.random() * (this.canvasHeight / this.gridSize)) * this.gridSize
      }
    },
    changeDirection(event) {
      const newDirection = event.key.replace('Arrow', '').toUpperCase()
      const oppositeDirection = {
        'RIGHT': 'LEFT',
        'LEFT': 'RIGHT',
        'UP': 'DOWN',
        'DOWN': 'UP'
      }
      if (this.direction !== oppositeDirection[newDirection]) {
        this.nextDirection = newDirection
      }
    },
    endGame() {
      clearInterval(this.gameInterval)
      this.gameCompleted = true
    },
    restartGame() {
      this.snake = [{ x: 160, y: 160 }]
      this.food = { x: 100, y: 100 }
      this.direction = 'RIGHT'
      this.nextDirection = 'RIGHT'
      this.score = 0
      this.gameCompleted = false
      this.initGame()
    },
    checkDifficulty() {
      // Menambah kecepatan permainan setiap kali skor mencapai threshold
      if (this.score > 0 && this.score % this.difficultyIncreaseThreshold === 0) {
        this.increaseDifficulty()
      }
    },
    increaseDifficulty() {
      clearInterval(this.gameInterval)
      this.intervalTime = Math.max(this.intervalTime - 10, 100) // Mengurangi interval, tapi tidak kurang dari 100 ms
      this.initGame()
    }
  }
}
</script>

<style scoped>
.snake-game-container {
  max-width: 100%;
}

canvas {
  display: block;
  margin: 0 auto;
  background-color: black;
  border-radius: 8px;
}

h1 {
  color: #1d4ed8; /* Tailwind Blue-600 */
}

.text-center {
  text-align: center;
}
</style>
