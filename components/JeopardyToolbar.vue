<template>
  <div class="flex items-center justify-between mb-6 flex-wrap gap-2">
    <h1 class="text-3xl font-extrabold">Jeopardy</h1>

    <div class="flex gap-2 flex-wrap">
      <button
        @click="$emit('toggleMode')"
        class="px-4 py-2 rounded shadow bg-indigo-600 text-white"
        aria-label="Switch edit present">
        {{ mode === 'edit' ? 'Switch to Presentation' : 'Switch to Edit' }}
      </button>

      <button
        @click="$emit('reset')"
        class="px-4 py-2 rounded shadow bg-red-500 text-white"
        aria-label="Reset board">
        Reset
      </button>

      <button
        @click="$emit('export')"
        class="px-4 py-2 rounded shadow bg-green-600 text-white"
        aria-label="Export json">
        Export JSON
      </button>

      <label
        class="px-4 py-2 rounded shadow bg-yellow-500 text-white cursor-pointer"
      >
        Import JSON
        <input
          type="file"
          accept="application/json"
          @change="$emit('import', $event)"
          class="hidden"
        />
      </label>
    </div>
  </div>
</template>

<script setup>
defineProps({
  mode: String,
})
defineEmits(['toggleMode', 'reset', 'export', 'import'])
</script>
