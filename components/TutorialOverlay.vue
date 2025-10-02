<template>
  <div v-if="visible" class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4">
    <div class="bg-white rounded-lg shadow-lg max-w-xl w-full p-6 text-center">
      <h2 class="text-2xl font-bold mb-4">Welcome to Jeopardy!</h2>
      <p class="mb-4">
        Click Add Category button to add as many categories as needed. Edit the questions
        by clicking each tile. Add players with the Add Player field. Finally switch over
        to presentation mode when ready. It is possible to export the questions as json and
        load back in anytime, anywhere.
      </p>
      <button 
        @click="close"
        class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Got it!
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  localStorageKey: { type: String, default: 'jeopardyTutorialSeen' }
})
const emit = defineEmits(['close'])

const visible = ref(false)

onMounted(() => {
  const seen = localStorage.getItem(props.localStorageKey)
  if (!seen) visible.value = true
})

function close() {
  visible.value = false
  localStorage.setItem(props.localStorageKey, 'true')
  emit('close')
}
</script>
