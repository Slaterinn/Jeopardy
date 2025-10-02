<template>
  <transition name="fade">
    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/95 p-4"
    >
      <div class="flex flex-col items-center justify-center w-full h-full p-8 text-center">
        <!-- Points -->
        <h2 class="text-[6vw] sm:text-[5vw] font-bold text-white mb-6">
          {{ question?.value }} points
        </h2>

        <!-- Question -->
        <p class="text-[8vw] sm:text-[6vw] font-extrabold text-white break-words leading-tight">
          {{ question?.question }}
        </p>

        <!-- Answer -->
        <p
          v-if="question?.revealed"
          class="mt-8 text-[6vw] sm:text-[5vw] italic text-gray-300 break-words leading-snug"
        >
          Answer: {{ question.answer }}
        </p>

        <!-- Buttons -->
        <div class="flex justify-center space-x-8 mt-12">
          <!-- Reveal Answer -->
          <button
            v-if="!question?.revealed"
            @click="reveal"
            class="bg-green-500 text-white px-8 py-4 rounded text-3xl hover:bg-green-600"
            aria-label="Reveal question">
            Reveal Answer
          </button>

          <!-- Toggle Taken -->
          <button
            v-if="!question?.revealed || question?.revealed"
            @click="$emit('toggle-taken')"
            :class="[
              'px-8 py-4 rounded text-3xl',
              question?.revealed || question?.taken
                ? 'bg-indigo-700 text-white hover:bg-indigo-800'
                : 'bg-indigo-600 text-white hover:bg-indigo-700'
            ]"
            aria-label="Toggle taken">
            {{ question?.taken ? 'Unmark Taken' : 'Mark Taken' }}
          </button>

          <!-- Cancel -->
          <button
            @click="$emit('close')"
            class="bg-gray-500 text-white px-8 py-4 rounded text-3xl hover:bg-gray-600"
            aria-label="Go Back">
            Back
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup>
const props = defineProps({
  visible: Boolean,
  question: Object,
})

const emit = defineEmits(['close', 'reveal', 'toggle-taken'])

function reveal() {
  // Reveal the answer and automatically mark taken
  emit('reveal')
  emit('toggle-taken')
}
</script>
