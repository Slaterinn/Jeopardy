<template>
  <div
    v-if="visible"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50"
  >
    <div class="bg-white rounded-lg shadow-lg p-6 max-w-md w-full text-center">
      <h2 class="text-xl font-bold mb-4">{{ title }}</h2>
      <p class="mb-6">{{ message }}</p>

      <div class="flex justify-center gap-4">
        <button
          @click="$emit('cancel')"
          class="px-6 py-2 bg-gray-300 rounded hover:bg-gray-400"
          aria-label="Cancel">
          Cancel
        </button>
        <button
          @click="$emit('confirm')"
          class="px-6 py-2 bg-red-600 text-white rounded hover:bg-red-700"
          aria-label="Confirm">
          Confirm
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  visible: Boolean,
  title: { type: String, default: 'Confirm' },
  message: { type: String, default: 'Are you sure?' },
})
defineEmits(['confirm', 'cancel'])
</script>
