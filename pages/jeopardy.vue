<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="max-w-7xl mx-auto space-y-6">
      <JeopardyToolbar :mode="mode" @toggleMode="toggleMode" />
      <JeopardyEditor v-if="mode === 'edit'" v-model:board="board" />
      <JeopardyBoard v-else v-model:board="board" />
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import JeopardyToolbar from '~/components/JeopardyToolbar.vue'
import JeopardyEditor from '~/components/JeopardyEditor.vue'
import JeopardyBoard from '~/components/JeopardyBoard.vue'

const mode = ref('edit')
const board = reactive({
  categories: []
})

function toggleMode() {
  mode.value = mode.value === 'edit' ? 'present' : 'edit'
}
</script>
