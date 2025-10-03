<template>
  <div
    v-if="visible"
    class="fixed inset-0 flex items-center justify-center bg-black/50 z-50 p-4"
  >
    <div class="absolute inset-0 bg-black/70" @click="$emit('close')"></div>
    <div class="bg-white rounded-lg shadow-lg max-w-2xl w-full p-8 relative z-10 text-center space-y-6">
      <h3 class="text-2xl font-bold">
        {{ question.category }} — {{ question.value }}
      </h3>

      <!-- Question text -->
      <div class="text-xl text-gray-900">
        {{ question.question || "No question yet" }}
      </div>

      <!-- Optional image for the question -->
      <div v-if="question.imageUrl" class="my-4">
        <img 
          :src="question.imageUrl" 
          alt="Question image" 
          class="max-w-full max-h-60 mx-auto rounded shadow-lg"
        />
      </div>

      <!-- Answer with transition -->
      <transition name="reveal-fade">
        <div
          v-if="question.revealed"
          class="mt-6 bg-green-100 border-l-4 border-green-500 p-4 rounded-lg text-green-800 text-2xl font-extrabold"
        >
          {{ question.answer || "No answer yet" }}
        </div>
      </transition>

      <!-- Buttons -->
      <div class="flex justify-center gap-4 mt-8">
        <button
          @click="$emit('close')"
          class="px-6 py-3 bg-gray-400 text-white rounded-lg hover:bg-gray-500"
        >
          Back
        </button>

        <button
          v-if="!question.revealed"
          @click="$emit('reveal')"
          class="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
        >
          Reveal Answer
        </button>

        <button
          @click="$emit('toggle-taken')"
          :class="question.taken ? 'bg-purple-800' : 'bg-purple-600'"
          class="px-6 py-3 text-white rounded-lg hover:bg-purple-700"
        >
          {{ question.taken ? 'Unmark Taken' : 'Mark Taken' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  visible: Boolean,
  question: {
    type: Object,
    default: () => ({})
  }
})
const emit = defineEmits(['close', 'reveal', 'toggle-taken'])
</script>
