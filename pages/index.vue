<script setup>
import { ref, watch, onMounted } from 'vue'

// Jeopardy game components
import JeopardyToolbar from '~/components/JeopardyToolbar.vue'
import JeopardyEditor from '~/components/JeopardyEditor.vue'
import JeopardyBoard from '~/components/JeopardyBoard.vue'

// Scoreboard / player components
import ScoreboardChart from '~/components/ScoreboardChart.vue'
import PlayerManager from '~/components/PlayerManager.vue'
import ScoreControls from '~/components/ScoreControls.vue'

const mode = ref('edit')

// Always initialize a reactive object
const board = ref({ categories: [] })
const teams = ref([])
const players = ref([])

// Load from localStorage on mount
onMounted(() => {
  const savedBoard = localStorage.getItem('jeopardyBoard')
  if (savedBoard) board.value = JSON.parse(savedBoard)

  const savedTeams = localStorage.getItem('jeopardyTeams')
  if (savedTeams) teams.value = JSON.parse(savedTeams)
})

// Persist to localStorage automatically (deep watch)
watch(
  board,
  (newBoard) =>
    localStorage.setItem('jeopardyBoard', JSON.stringify(newBoard)),
  { deep: true }
)

watch(
  teams,
  (newTeams) =>
    localStorage.setItem('jeopardyTeams', JSON.stringify(newTeams)),
  { deep: true }
)

function toggleMode() {
  mode.value = mode.value === 'edit' ? 'presentation' : 'edit'
}

// Reset board safely without reassigning the ref directly
function resetBoard() {
  const newBoard = {
    ...board.value,
    categories: board.value.categories.map(cat => ({
      ...cat,
      questions: cat.questions.map(q => ({ ...q, revealed: false, taken: false }))
    }))
  }
  board.value = newBoard

  teams.value = teams.value.map(t => ({ ...t, score: 0 }))
}

const showResetModal = ref(false)

function confirmReset() {
  resetBoard()
  showResetModal.value = false
}

// Export JSON
function exportBoard() {
  const blob = new Blob(
    [JSON.stringify({ board: board.value, teams: teams.value }, null, 2)],
    { type: 'application/json' }
  )
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = 'jeopardy-quiz.json'
  link.click()
}

// Import JSON
function importBoard(event) {
  const file = event.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      if (data.board) board.value = { ...data.board }
      if (data.teams) teams.value = data.teams.map(t => ({ ...t }))
    } catch {
      alert('Invalid JSON file')
    }
  }
  reader.readAsText(file)
}
</script>

<template>
  <div class="p-6 max-w-7xl mx-auto">
    <JeopardyToolbar
      :mode="mode"
      @toggleMode="toggleMode"
      @reset="showResetModal = true"
      @export="exportBoard"
      @import="importBoard"
    />

    <ConfirmModal
      :visible="showResetModal"
      title="Reset Board?"
      message="This will unmark all tiles and reset scores. Are you sure?"
      @confirm="confirmReset"
      @cancel="showResetModal = false"
    />

    <div v-if="mode === 'edit'">
      <JeopardyEditor v-model:board="board" />
    </div>
    <div v-else>
      <JeopardyBoard v-model:board="board" />
    </div>

    <PlayerManager v-model:players="players" />
    <ScoreboardChart :players="players" />
    <ScoreControls v-model:players="players" />
  </div>
</template>
