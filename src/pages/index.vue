<script setup lang="ts">
import { dirxml } from 'console'
import { blockStatement } from '@babel/types'

interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine: boolean
  flagged: boolean
  adjacentMines: number
}

const WIDTH = 10
const HEIGHT = 10
const state = reactive(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from(
      { length: WIDTH },
      (_, x): BlockState => ({ x, y, adjacentMines: 0 }),
    ),
  ),
)

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
  [0, -1],
]
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-fuchsia-500',
  'text-pink-500',
  'text-orange-500',
  'text-white-500',
]

function generateMines(initial: BlockState) {
  for (const row of state)
    for (const block of row) block.mine = Math.random() < 0.1

  updateNumbers()
}

function updateNumbers() {
  state.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return 1
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function expendZero(block: BlockState) {
  if (block.adjacentMines !== 0 || block.mine)
    return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

let mineGenerated = false
const dev = false

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  expendZero(block)
}

function getBlockClass(block: BlockState) {
  if (!block.revealed)
    return 'bg-gray-500/10'
  return block.mine ? 'text-red' : numberColors[block.adjacentMines]
}
</script>

<template>
  <div>
    Mineseeper
    <div p1 />
    <div
      v-for="(row, y) in state"
      :key="y"
      flex="~"
      justify-center
      items-center
    >
      <button
        v-for="(block, x) in row"
        :key="x"
        w-10
        h-10
        m1
        border
        hover:bg-gray
        :class="getBlockClass(block)"
        flex="~"
        justify-center
        items-center
        @click="onClick(block)"
      >
        <template v-if="block.revealed">
          <div v-if="block.mine">
            x
          </div>
          <div v-else>
            {{ block.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
  </div>
</template>
