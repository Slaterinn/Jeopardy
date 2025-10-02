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
      :style="`grid-template-columns: repeat(${localCategories.length}, 1fr)`"
    >
      <!-- Category headers -->
      <div
        v-for="(cat, ci) in localCategories"
        :key="'header-' + ci"
        class="bg-blue-800 text-white text-center font-bold text-xl p-4 rounded relative"
      >
        <input
          v-model="cat.title"
          class="bg-transparent text-center w-full font-bold text-xl border-b border-white focus:outline-none"
          placeholder="Category"
        />
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
          v-for="(cat, ci) in localCategories"
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
          Edit: {{ localCategories[activeQuestion.ci]?.title || 'Unknown Category' }} —
          {{ activeQuestion.value }}
        </h3>
        <div class="mb-4">
          <label class="block font-semibold mb-1">Question</label>
          <textarea
            v-model="localQuestion.question"
            rows="3"
            class="w-full border rounded p-2"
            placeholder="Enter question..."
          ></textarea>
        </div>
        <div class="mb-4">
          <label class="block font-semibold mb-1">Answer</label>
          <input
            v-model="localQuestion.answer"
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
            @click="saveQuestion"
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
import { ref, computed, reactive, watch } from 'vue'

const props = defineProps({
  board: { type: Object, required: true },
})
const emit = defineEmits(['update:board'])

const activeQuestion = ref(null)
const localQuestion = reactive({ question: '', answer: '' })

/* Local reactive copy of categories
const localCategories = reactive(
  props.board.categories.map(cat => ({
    title: cat.title,
    questions: cat.questions.map(q => ({ ...q }))
  }))
)*/
const localCategories = reactive([])

// sync localCategories when board prop changes
watch(
  () => props.board.categories,
  (newCats) => {
    localCategories.length = 0
    newCats.forEach(cat => {
      localCategories.push({
        title: cat.title,
        questions: cat.questions.map(q => ({ ...q }))
      })
    })
  },
  { immediate: true, deep: true }
)

const maxRows = computed(() =>
  Math.max(...localCategories.map(c => c.questions.length), 5)
)

function addCategory() {
  const newCategory = {
    title: 'New Category',
    questions: [100, 200, 300, 400, 500].map(v => ({
      value: v,
      question: '',
      answer: '',
      revealed: false
    }))
  }

  localCategories.push(newCategory)  // update local copy

  const newBoard = {
    ...props.board,
    categories: [...localCategories.map(cat => ({
      ...cat,
      questions: cat.questions.map(q => ({ ...q }))
    }))]
  }

  emit('update:board', newBoard)
}

function deleteCategory(ci) {
  const newBoard = {
    ...props.board,
    categories: props.board.categories.filter((_, i) => i !== ci)
  }
  emit('update:board', newBoard)
}

function addQuestion(ci, qi) {
  const newBoard = {
    ...props.board,
    categories: props.board.categories.map((cat, cIdx) => {
      if (cIdx !== ci) return cat
      const newQuestions = [...cat.questions]
      newQuestions[qi] = {
        value: (qi + 1) * 100,
        question: '',
        answer: '',
        revealed: false
      }
      return { ...cat, questions: newQuestions }
    })
  }
  emit('update:board', newBoard)
}

function openEditor(ci, qi) {
  const category = localCategories[ci]
  if (!category || !category.questions[qi]) return
  activeQuestion.value = { ci, qi, value: category.questions[qi].value }
  localQuestion.question = category.questions[qi].question
  localQuestion.answer = category.questions[qi].answer
}

function closeEditor() {
  activeQuestion.value = null
}

function saveQuestion() {
  if (!activeQuestion.value) return
  const ci = activeQuestion.value.ci
  const qi = activeQuestion.value.qi

  const newBoard = {
    ...props.board,
    categories: props.board.categories.map((cat, cIdx) => {
      if (cIdx !== ci) return cat
      const newQuestions = [...cat.questions]
      newQuestions[qi] = {
        ...newQuestions[qi],
        question: localQuestion.question,
        answer: localQuestion.answer
      }
      return { ...cat, questions: newQuestions }
    })
  }
  emit('update:board', newBoard)
  closeEditor()
}
</script>
