<script setup>
import { ref, watchEffect } from 'vue'
import { useMagicKeys, useSwipe } from '@vueuse/core'


const grid = 30
const snake = ref([{ x: 15, y: 15 }])
const food = ref(randomFood())
const direction = ref('left')
let gameInterval
const speed = ref(200)
const { arrowleft, arrowright, arrowup, arrowdown, space } = useMagicKeys()
const score = ref(0)
const swipeFild = ref(null)
const { direction: swipeDirection } = useSwipe(swipeFild)
const gameOver = ref(false)

function isCellSnake(x, y) {
    let result = false
    snake.value.forEach((segment) => {
        if (x === segment.x && y === segment.y) {
            result = true
        }
    })
    return result
}

function isCellFood(x, y) {
    if (x === food.value.x && y === food.value.y) {
        return true
    }
}

function randomFood() {
    const x = Math.floor(Math.random() * grid)
    const y = Math.floor(Math.random() * grid)
    return { x, y }
}


function move() {
    const head = { ...snake.value[0] }
    switch (direction.value) {
        case 'up':
            head.y--
            break
        case 'down':
            head.y++
            break
        case 'left':
            head.x--
            break
        case 'right':
            head.x++
            break
    }

    if (checkCollision(head)) {
        stopGame()
        return
    }

    snake.value.unshift(head)

    if (head.x === food.value.x && head.y === food.value.y) {
        food.value = randomFood()
        updateScore()
        speed.value = speed.value - 5
        moving()
    } else {
        snake.value.pop()
    }
}

function startGame() {
    snake.value = [{ x: 10, y: 10 }]
    food.value = randomFood()
    direction.value = 'down'
    speed.value = 200
    gameOver.value = false
    updateScore()
    moving()
}

function moving() {
    clearInterval(gameInterval)
    gameInterval = setInterval(() => {
        move()
    }, speed.value)
}

function stopGame() {
    clearInterval(gameInterval)
    gameOver.value = true
}

function checkCollision(head) {
    if (head.x < 0 || head.x > grid - 1 || head.y < 0 || head.y > grid - 1) {
        return true
    }

    for (let i = 1; i < snake.value.length; i++) {
        if (head.x === snake.value[i].x && head.y === snake.value[i].y) {
            return true
        }
    }
}


watchEffect(() => {
    if (arrowdown.value && (direction.value !== 'up' || snake.value.length === 1)) {
        direction.value = 'down'
    }
    if (arrowup.value && (direction.value !== 'down' || snake.value.length === 1)) {
        direction.value = 'up'
    }
    if (arrowright.value && (direction.value !== 'left' || snake.value.length === 1)) {
        direction.value = 'right'
    }
    if (arrowleft.value && (direction.value !== 'right' || snake.value.length === 1)) {
        direction.value = 'left'
    }
    if (space.value) {
        startGame()
    }
})

watchEffect(() => {
    if (swipeDirection.value === 'up' && (direction.value !== 'down' || snake.value.length === 1)) {
        direction.value = 'up'
    }
    if (swipeDirection.value === 'down' && (direction.value !== 'up' || snake.value.length === 1)) {
        direction.value = 'down'
    }
    if (swipeDirection.value === 'right' && (direction.value !== 'left' || snake.value.length === 1)) {
        direction.value = 'right'
    }
    if (swipeDirection.value === 'left' && (direction.value !== 'right' || snake.value.length === 1)) {
        direction.value = 'left'
    }
})

document.addEventListener('touchmove', function (event) {
    event.preventDefault()
}, { passive: false })

function updateScore() {
    score.value = snake.value.length - 1
}

</script>

<template>
    <div class="main" ref="swipeFild">
        <div class="game__stats">Рахунок: {{ score }}</div>
        <div class="game">
            <div class="game__board">
                <div v-for="(row, y) in grid" :key="y" class="game__row">
                    <div v-for="(cell, x) in grid" :key="x" class="game__cell"
                        :class="{ 'game__snake': isCellSnake(x, y), 'game__food': isCellFood(x, y) }"></div>
                </div>
                <div v-if="gameOver" class="game__text">Ви програли</div>
            </div>
        </div>

        <button class="game__btn" @click="startGame">Почати гру</button>
    </div>
</template>

<style scoped lang="scss">
.main {
    padding: 30px 15px 0;
    min-height: 100vh;
    font-family: Arial, Helvetica, sans-serif;
}

.game {
    max-width: 400px;
    border: 5px solid #b4a7a7;
    margin: 0 auto;
    position: relative;

    &__stats {
        text-align: center;
        padding-bottom: 20px;
    }

    &__board {}

    &__snake {
        background-color: rgb(98, 167, 98);
    }

    &__food {
        background-color: rgb(223, 54, 54);
    }

    &__btn {
        display: flex;
        margin: 20px auto 0;
        padding: 10px;
        background-color: #b4a7a7;
        border: none;
        border-radius: 30px;
        font-size: 20px;
        cursor: pointer;
        outline: none;
    }

    &__row {
        display: flex;

        &+& {
            border-top: 1px solid #b4a7a7;
        }
    }

    &__cell {
        display: flex;
        aspect-ratio: 1 / 1;
        width: 100%;

        &+& {
            border-left: 1px solid #b4a7a7;
        }

    }

    &__text {
        position: absolute;
        color: rgb(151, 62, 62);
        font-size: 32px;
        text-align: center;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        
    }
}
</style>
