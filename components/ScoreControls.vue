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
  <div class="grid grid-cols-3 gap-4 mt-4">
    <div v-for="(player, index) in players" :key="index" class="flex flex-col items-center space-y-2">
      <span class="font-bold">{{ player.name }}</span>
      <div class="flex space-x-2">
        <button @click="adjustScore(index, -100)" class="bg-red-500 text-white px-3 py-1 rounded">-100</button>
        <span class="px-3 py-1 font-bold">{{ player.score }}</span>
        <button @click="adjustScore(index, 100)" class="bg-green-500 text-white px-3 py-1 rounded">+100</button>
      </div>
    </div>
  </div>
</template>
