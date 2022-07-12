<script setup lang="ts">
import MineBlock from '~/components/MineBlock.vue'
import type { BlockState } from '~/types/BlockState'

let mineGenerated = false
const dev = false

const WIDTH = 5
const HEIGHT = 5
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

function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (dev)
        block.revealed = true
      block.mine = Math.random() < 0.1
    }
  }

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
  return directions
    .map(([dx, dy]) => {
      const x2 = block.x + dx
      const y2 = block.y + dy
      if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
        return undefined
      return state[y2][x2]
    })
    .filter(Boolean) as BlockState[]
}

function expendZero(block: BlockState) {
  if (block.adjacentMines !== 0 || block.mine || block.flagged)
    return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
  checkGameState()
}
function onClick(block: BlockState) {
  checkGameState()

  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  if (block.flagged)
    return
  if (block.mine)
    alert('Boom')
  block.revealed = true
  expendZero(block)
}

function checkGameState() {
  const blocks = state.flat()
  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You cheat')
    else alert('you win')
  }
}
</script>

<template>
  <div>
    Mineseeper
    <div p1 />
    <button>toggle dev</button>
    <div
      v-for="(row, y) in state"
      :key="y"
      flex="~"
      justify-center
      items-center
      @contextmenu.prevent=""
    >
      <MineBlock
        v-for="(block, x) in row"
        :key="x"
        :block="block"
        @click="onClick(block)"
        @contextmenu.prevent="onRightClick(block)"
      />
    </div>
  </div>
</template>
