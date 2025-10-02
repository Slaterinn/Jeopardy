<script setup>
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

const props = defineProps({
  players: {
    type: Array,
    default: () => []  // ✅ Safe fallback
  }
})

const chartData = computed(() => ({
  labels: props.players.map(p => p.name),
  datasets: [
    {
      label: 'Score',
      data: props.players.map(p => p.score),
      backgroundColor: ['#3b82f6', '#f97316', '#10b981', '#e11d48']
    }
  ]
}))

const chartOptions = {
  responsive: true,
  plugins: {
    legend: { display: false }
  },
  scales: {
    y: { beginAtZero: true }
  }
}
</script>

<template>
  <div class="mt-8 space-y-4">
    <Bar
      v-if="players.length"
      :data="chartData"
      :options="chartOptions"
      class="h-[20vh]" 
    />
    <p v-else class="text-gray-500 text-center">No players yet. Add some!</p>
  </div>
</template>
