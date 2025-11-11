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
      <div class="text-xl text-gray-900 whitespace-pre-line">
        {{ question.question || "No question yet" }}
      </div>

      <!-- Optional image for the question -->
      <div v-if="question.imageUrl" class="my-4 overflow-auto">
        <img 
          :src="question.imageUrl" 
          alt="Question image" 
          loading="lazy"
          class="max-w-full max-h-80 mx-auto rounded shadow-lg"
        />
      </div>

      <!-- Sound player -->
      <audio
        v-if="question.soundUrl"
        :src="question.soundUrl"
        controls
        class="mt-4 w-full"
      />

      <!-- YouTube video -->
      <div v-if="question.youtubeUrl" class="flex flex-col items-center mt-6">
        <div v-if="!showVideo">
          <button
            @click="playVideo"
            class="px-6 py-3 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 shadow-md transition"
          >
            ▶ Play Video
          </button>
        </div>

        <transition name="fade">
          <div
            v-if="showVideo"
            class="relative w-full max-w-2xl aspect-video mt-4 rounded-lg overflow-hidden shadow-lg"
          >
            <iframe
              :src="youtubeEmbedUrl"
              frameborder="0"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen
              class="w-full h-full"
            ></iframe>

            <button
              @click="closeVideo"
              class="absolute top-2 right-2 bg-indigo-700 bg-opacity-80 text-white rounded-full w-8 h-8 flex items-center justify-center hover:bg-indigo-800 transition"
            >
              ✕
            </button>
          </div>
        </transition>
      </div>


      <!-- Answer with transition -->
      <transition name="reveal-fade">
        <div
          v-if="question.revealed"
          class="mt-6 bg-green-100 border-l-4 border-green-500 p-4 rounded-lg text-green-800 text-2xl font-extrabold whitespace-pre-line"
        >
          {{ question.answer || "No answer yet" }}
        </div>
      </transition>

      <!-- Buttons -->
      <div class="flex justify-center gap-4 mt-8">
        <button
          @click="$emit('close')"
          class="px-6 py-3 bg-indigo-300 text-white rounded-lg hover:bg-indigo-400"
        >
          Back
        </button>

        <button
          v-if="!question.revealed"
          @click="$emit('reveal')"
          class="px-6 py-3 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700"
        >
          Reveal Answer
        </button>

        <button
          @click="$emit('toggle-taken')"
          :class="question.taken ? 'bg-indigo-800' : 'bg-indigo-600'"
          class="px-6 py-3 text-white rounded-lg hover:bg-indigo-700"
        >
          {{ question.taken ? 'Unmark Taken' : 'Mark Taken' }}
        </button>
      </div>

      <!-- Timer Section -->
      <div class="mt-6 text-center">
        <!-- Timer Display -->
        <div
          v-if="timer !== null"
          class="text-4xl font-extrabold text-red-600 drop-shadow-lg transition-all duration-300"
          :class="{ 'animate-pulse': timerActive }"
        >
          {{ timer }}
        </div>

        <!-- Timer Controls -->
        <div class="mt-3">
          <button
            v-if="!timerActive && timer === null"
            @click="startTimer()"
            class="px-4 py-2 text-sm bg-indigo-600 text-white rounded-md hover:bg-indigo-700"
          >
            Start Timer
          </button>

          <button
            v-else-if="timerActive"
            @click="stopTimer()"
            class="px-4 py-2 text-sm bg-red-500 text-white rounded-md hover:bg-red-600"
          >
            Stop Timer
          </button>

          <button
            v-else-if="!timerActive && timer !== null"
            @click="resetTimer()"
            class="px-3 py-1 text-xs bg-gray-400 text-white rounded-md hover:bg-gray-500"
          >
            Reset Timer
          </button>
        </div>
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


//Timer state
import { ref, computed  } from 'vue'

const timer = ref(null)
const timerActive = ref(false)
let interval = null

// Define audio elements
const timerSound = new Audio('/sounds/jeopardy-countdown.mp3') // looping countdown sound
timerSound.loop = true

const finalSound = new Audio('/sounds/timer-end.mp3') // sound at timer end

function startTimer() {
  timer.value = 5 // adjust duration here
  timerActive.value = true
  clearInterval(interval)

  // Start countdown sound
  timerSound.currentTime = 0
  timerSound.play().catch(() => {
    console.warn('Autoplay blocked: user interaction needed first')
  })

  interval = setInterval(() => {
    if (timer.value > 0) {
      timer.value--
    } else {
      clearInterval(interval)
      timerActive.value = false

      // Stop countdown sound
      timerSound.pause()
      timerSound.currentTime = 0

      // Play final sound
      finalSound.currentTime = 0
      finalSound.play().catch(() => {
        console.warn('Autoplay blocked: user interaction needed first')
      })

      // Dramatic flash
      const flash = document.createElement('div')
      flash.className = 'screen-flash'
      document.body.appendChild(flash)
      setTimeout(() => flash.remove(), 600)
    }
  }, 1000)
}



function stopTimer() {
  clearInterval(interval)
  timerActive.value = false
  // Stop countdown sound
  timerSound.pause()
  timerSound.currentTime = 0
}

function resetTimer() {
  clearInterval(interval)
  timer.value = null
  timerActive.value = false
}

//Video elements
const showVideo = ref(false)

const youtubeEmbedUrl = computed(() => {
  const url = props.question.youtubeUrl
  if (!url) return ''
  const match = url.match(/(?:youtu\.be\/|youtube\.com\/(?:watch\?v=|embed\/|v\/))([\w-]+)/)
  const id = match ? match[1] : ''
  return id ? `https://www.youtube.com/embed/${id}?autoplay=1` : ''
})

function playVideo() {
  showVideo.value = true
}

function closeVideo() {
  showVideo.value = false
}

</script>
