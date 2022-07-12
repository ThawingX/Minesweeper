<script setup lang='ts'>
import type { BlockState } from '~/types/BlockState'
defineProps <{ block: BlockState }>()
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

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-grap-500/10'
  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray'
  return block.mine ? 'text-red' : numberColors[block.adjacentMines]
}
</script>

<template>
  <button
    w-10
    h-10
    m1
    border
    :class="getBlockClass(block)"
    flex="~"
    justify-center
    items-center
  >
    <template v-if="block.flagged">
      <div>T</div>
    </template>
    <template v-else-if="block.revealed">
      <div v-if="block.mine">
        x
      </div>
      <div v-else>
        {{ block.adjacentMines }}
      </div>
    </template>
  </button>
</template>
