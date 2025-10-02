<script setup>
const props = defineProps({
  players: Array
})
const emit = defineEmits(['update:players'])

const newPlayer = ref('')

function addPlayer() {
  if (!newPlayer.value) return
  emit('update:players', [...props.players, { name: newPlayer.value, score: 0 }])
  newPlayer.value = ''
}

function removePlayer(index) {
  const updated = [...props.players]
  updated.splice(index, 1)
  emit('update:players', updated)
}
</script>

<template>
  <div class="flex flex-col sm:flex-row items-center gap-2 mb-4 mt-16">
  <input
    v-model="newPlayer"
    placeholder="Player name"
    class="border rounded px-2 py-1 flex-1 w-full sm:w-auto"
    @keyup.enter="addPlayer"
  />

  <button
    @click="addPlayer"
    class="bg-blue-500 text-white px-4 py-1 rounded self-stretch w-full sm:w-auto"
  >
    Add Player
  </button>
</div>

  <div class="flex gap-2 flex-wrap">
    <div v-for="(player, index) in players" :key="index" class="bg-gray-100 px-3 py-1 rounded flex items-center gap-2">
      <span>{{ player.name }}</span>
      <button @click="removePlayer(index)" class="text-red-500 font-bold">✕</button>
    </div>
  </div>
</template>
