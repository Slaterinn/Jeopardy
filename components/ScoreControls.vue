<script setup>
const props = defineProps({
  players: Array
})
const emit = defineEmits(['update:players'])

function adjustScore(index, amount) {
  const updated = [...props.players]
  updated[index].score += amount
  emit('update:players', updated)
}
</script>

<template>
  <div class="flex flex-wrap gap-4 mt-4 justify-center">
    <div
      v-for="(player, index) in players"
      :key="index"
      class="flex flex-col items-center bg-gray-100 p-4 rounded shadow w-32"
    >
      <span class="font-bold text-lg">{{ player.name }}</span>
      <span class="text-xl mb-2">{{ player.score }}</span>

      <!-- Score buttons stacked vertically -->
      <div class="flex flex-col gap-2 w-full">
        <button
          @click="adjustScore(index, +100)"
          class="bg-green-500 text-white py-1 rounded hover:bg-green-600"
        >
          +100
        </button>
        <button
          @click="adjustScore(index, -100)"
          class="bg-red-500 text-white py-1 rounded hover:bg-red-600"
        >
          -100
        </button>
      </div>
    </div>
  </div>
</template>
