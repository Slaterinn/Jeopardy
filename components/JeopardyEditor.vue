<template>
  <div>
    <!-- Toolbar for categories -->
    <div class="flex gap-2 mb-4">
      <button
        @click="addCategory"
        class="px-4 py-2 bg-blue-600 text-white rounded cursor-pointer"
      >
        Add Category
      </button>
    </div>

    <!-- Board-style editor -->
    <div
      class="grid gap-2"
      :style="`grid-template-columns: repeat(${board.categories.length}, 1fr)`"
    >
      <!-- Category headers -->
      <div
        v-for="(cat, ci) in board.categories"
        :key="'header-' + ci"
        class="bg-blue-800 text-white text-center font-bold text-xl p-4 rounded relative"
      >
        <input
          v-model="cat.title"
          class="bg-transparent text-center w-full font-bold text-xl border-b border-white focus:outline-none"
          placeholder="Category"
        />

        <!-- Delete button (top right corner) -->
        <button
          @click="deleteCategory(ci)"
          class="absolute top-1 right-1 bg-red-600 text-white text-xs px-2 py-1 rounded hover:bg-red-700 cursor-pointer"
          title="Delete category"
        >
          ✕
        </button>
      </div>

      <!-- Question rows -->
      <template v-for="i in maxRows" :key="'row-' + i">
        <div
          v-for="(cat, ci) in board.categories"
          :key="'q-' + ci + '-' + i"
          class="flex items-center justify-center"
        >
          <button
            v-if="cat.questions[i - 1]"
            @click="openEditor(ci, i - 1)"
            class="w-full h-24 text-3xl font-bold flex items-center justify-center rounded bg-yellow-400 hover:bg-yellow-500 text-blue-900 shadow-lg"
          >
            {{ cat.questions[i - 1].value }}
          </button>
          <button
            v-else
            @click="addQuestion(ci, i - 1)"
            class="w-full h-24 bg-gray-200 text-gray-500 rounded"
          >
            +
          </button>
        </div>
      </template>
    </div>

    <!-- Question editor modal -->
    <div
      v-if="activeQuestion"
      class="fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div class="absolute inset-0 bg-black/70" @click="closeEditor"></div>
      <div class="bg-white max-w-lg w-full rounded shadow-lg p-6 z-10">
        <h3 class="text-xl font-bold mb-4">
          Edit: {{ board.categories[activeQuestion.ci].title }} —
          {{ activeQuestion.value }}
        </h3>
        <div class="mb-4">
          <label class="block font-semibold mb-1">Question</label>
          <textarea
            v-model="board.categories[activeQuestion.ci].questions[activeQuestion.qi].question"
            rows="3"
            class="w-full border rounded p-2"
            placeholder="Enter question..."
          ></textarea>
        </div>
        <div class="mb-4">
          <label class="block font-semibold mb-1">Answer</label>
          <input
            v-model="board.categories[activeQuestion.ci].questions[activeQuestion.qi].answer"
            class="w-full border rounded p-2"
            placeholder="Enter answer..."
          />
        </div>

        <div class="flex justify-end gap-2">
          <button
            @click="closeEditor"
            class="px-4 py-2 bg-gray-400 text-white rounded"
          >
            Cancel
          </button>
          <button
            @click="closeEditor"
            class="px-4 py-2 bg-blue-600 text-white rounded"
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  board: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:board'])

const activeQuestion = ref(null)

const maxRows = computed(() =>
  Math.max(...props.board.categories.map((c) => c.questions.length), 5)
)

function addCategory() {
  props.board.categories.push({
    title: 'New Category',
    questions: [
      { value: 100, question: '', answer: '', revealed: false },
      { value: 200, question: '', answer: '', revealed: false },
      { value: 300, question: '', answer: '', revealed: false },
      { value: 400, question: '', answer: '', revealed: false },
      { value: 500, question: '', answer: '', revealed: false },
    ],
  })
}

function addQuestion(ci, qi) {
  props.board.categories[ci].questions[qi] = {
    value: (qi + 1) * 100,
    question: '',
    answer: '',
    revealed: false,
  }
}

function openEditor(ci, qi) {
  activeQuestion.value = {
    ci,
    qi,
    value: props.board.categories[ci].questions[qi].value,
  }
}

function closeEditor() {
  activeQuestion.value = null
}

function deleteCategory(ci) {
  props.board.categories.splice(ci, 1)
}
</script>
