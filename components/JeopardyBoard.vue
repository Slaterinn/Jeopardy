<template>
  <!-- Board grid -->
  <div
    class="grid gap-2 sm:gap-4"
    :style="`grid-template-columns: repeat(${board.categories.length}, minmax(120px, 1fr))`"
  >
    <!-- Category headers -->
    <div
      v-for="(cat, ci) in board.categories"
      :key="'header-' + ci"
      class="category-header"
    >
      {{ cat.title }}
    </div>

    <!-- Question rows -->
    <template v-for="rowIndex in maxRows" :key="'row-' + rowIndex">
      <div
        v-for="(cat, ci) in board.categories"
        :key="'q-' + ci + '-' + rowIndex"
        class="perspective"
      >
        <div
          v-if="cat.questions[rowIndex - 1]"
          class="relative w-full h-32 transition-transform duration-700 transform"
          :class="{
            'rotate-y-180': isRevealed(ci, rowIndex - 1)
          }"
        >
          <!-- Front / normal tile -->
          <button
            v-if="!cat.questions[rowIndex - 1].taken"
            @click="openQuestion(ci, rowIndex - 1)"
            class="tile-front w-full h-full"
            aria-label="Open question">
            {{ cat.questions[rowIndex - 1].value }}
          </button>

          <!-- Taken / gray tile -->
          <div
            v-else
            class="tile-taken w-full h-full flex items-center justify-center font-bold text-3xl"
          >
            {{ cat.questions[rowIndex - 1].value }}
          </div>

          <!-- Back of tile for flip (question revealed) -->
          <div
            v-if="activeQuestion && activeQuestion.ci === ci && activeQuestion.qi === rowIndex - 1"
            class="tile-back w-full h-full flex items-center justify-center p-2 text-center"
          >
            {{ activeQuestion.question || 'No question' }}
          </div>
        </div>
      </div>
    </template>
  </div>

  <!-- Full-screen Question Modal -->
  <JeopardyQuestionModal
    :visible="!!activeQuestion"
    :question="activeQuestion"
    @close="closeModal"
    @reveal="revealAnswer"
    @toggle-taken="toggleQuestionTaken"
  />
</template>

<script setup>
import { ref, computed } from 'vue'
import JeopardyQuestionModal from '~/components/JeopardyQuestionModal.vue'

const props = defineProps({
  board: { type: Object, required: true },
})
const emit = defineEmits(['update:board'])

const activeQuestion = ref(null)

// Compute max rows for layout
const maxRows = computed(() =>
  Math.max(...props.board.categories.map(c => c.questions.length))
)

// Open modal for a question
function openQuestion(ci, qi) {
  const q = props.board.categories[ci].questions[qi]
  activeQuestion.value = {
    ...q,
    ci,
    qi,
    category: props.board.categories[ci].title,
  }
}

// Close modal
function closeModal() {
  activeQuestion.value = null
}

// Reveal answer (also marks taken automatically)
function revealAnswer() {
  if (!activeQuestion.value) return
  activeQuestion.value.revealed = true
  toggleQuestionTaken(true)
}

// Toggle taken / unmark taken immutably
function toggleQuestionTaken(forceTaken = null) {
  if (!activeQuestion.value) return
  const { ci, qi } = activeQuestion.value
  const oldQ = props.board.categories[ci].questions[qi]

  const updatedQ = {
    ...oldQ,
    revealed: oldQ.revealed || activeQuestion.value.revealed,
    taken:
      forceTaken === true ? true :
      forceTaken === false ? false :
      !oldQ.taken
  }

  const newBoard = {
    ...props.board,
    categories: props.board.categories.map((cat, cIdx) => {
      if (cIdx !== ci) return cat
      return {
        ...cat,
        questions: cat.questions.map((q, qIdx) =>
          qIdx === qi ? updatedQ : q
        ),
      }
    }),
  }

  emit('update:board', newBoard)

  // Update local activeQuestion copy
  activeQuestion.value.taken = updatedQ.taken
  activeQuestion.value.revealed = updatedQ.revealed
}

// Helper to determine if a tile should show as revealed
function isRevealed(ci, qi) {
  return (
    activeQuestion.value &&
    activeQuestion.value.ci === ci &&
    activeQuestion.value.qi === qi &&
    activeQuestion.value.revealed
  )
}
</script>
