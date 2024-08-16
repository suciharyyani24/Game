<template>
    <div class="snake-game-container mx-auto mt-10 max-w-lg p-4">
        <canvas ref="gameCanvas" class="border border-gray-400 bg-black"></canvas>
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
            gameInterval: null,
            score: 0,
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
            }, 100)
        },
        drawGame(ctx) {
            ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)

            // Draw snake
            ctx.fillStyle = 'green'
            this.snake.forEach(segment => ctx.fillRect(segment.x, segment.y, this.gridSize, this.gridSize))

            // Draw food
            ctx.fillStyle = 'red'
            ctx.fillRect(this.food.x, this.food.y, this.gridSize, this.gridSize)
        },
        updateGame() {
            const head = { ...this.snake[0] }

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
                this.direction = newDirection
            }
        },
        endGame() {
            clearInterval(this.gameInterval)
            alert(`Game Over! Your score: ${this.score}`)
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
}
</style>
