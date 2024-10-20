<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { AlertCircleIcon } from 'lucide-vue-next'

defineProps<{ msg: string }>()

const leaderboard = ref([])
const loading = ref(true)
const error = ref<string | null>(null)

const emojis = ['🚀', '🤖', '🍺', '🦞', '🛸', '🪐', '👽', '🛠️', '💰', '🏆']

const futuramaNames = [
  'Fry', 'Leela', 'Bender', 'Professor Farnsworth', 'Amy', 'Hermes', 'Zoidberg', 'Nibbler',
  'Zapp Brannigan', 'Kif', 'Mom', 'Scruffy', 'Calculon', 'Robot Devil', 'Lrrr', 'Morbo',
  'Elzar', 'Flexo', 'Hedonismbot', 'Hypnotoad'
]

const fetchLeaderboard = async () => {
  loading.value = true
  error.value = null

  try {
    const response = await fetch('http://164.92.123.157:8282/leaderboard', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({})
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()

    if (data.error) {
      throw new Error(data.error)
    }

    leaderboard.value = data.leaderboard
  } catch (err) {
    console.error('Error fetching leaderboard:', err)
    error.value = `Great Spaghetti Monster! An error occurred: ${err}. Try again, meatbag!`
  } finally {
    loading.value = false
  }
}

const retryFetch = () => {
  fetchLeaderboard()
}

const formatENS = (ensAddress: string) => {
  return ensAddress.split('/').pop()
}

const formatBalance = (balance: string) => {
  return parseFloat(balance).toFixed(2)
}

const getRandomEmoji = () => {
  return emojis[Math.floor(Math.random() * emojis.length)]
}

const getFuturamaName = (originalName: string) => {
  const hash = originalName.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0)
  return futuramaNames[hash % futuramaNames.length]
}

onMounted(fetchLeaderboard)
</script>

<template>
    <div class="min-h-screen bg-gradient-to-br from-fuchsia-500 via-purple-600 to-indigo-500 flex items-center justify-center p-6 font-roboto-mono">
    <div class="w-full max-w-4xl bg-white rounded-xl shadow-2xl overflow-hidden relative">
      <div class="absolute inset-0 bg-gradient-to-tr from-yellow-300 via-pink-500 to-purple-600 opacity-20"></div>
      <div class="relative p-8">
        <h1 class="text-5xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-600 mb-2 text-center animate-pulse">
          {{ msg }}
        </h1>
        <p class="text-gray-700 text-center mb-8 text-lg">Good news, everyone! See who's leading the pack in our delivery universe!</p>

        <div v-if="loading" class="flex justify-center items-center py-12">
          <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-purple-500"></div>
        </div>

        <div v-else-if="error" class="text-center py-12">
          <AlertCircleIcon class="h-16 w-16 text-red-500 mx-auto mb-4 animate-bounce" />
          <p class="text-red-500 text-lg font-medium mb-4">{{ error }}</p>
          <button 
            @click="retryFetch" 
            class="bg-purple-600 text-white px-6 py-2 rounded-full hover:bg-purple-700 transition duration-300 ease-in-out transform hover:scale-105"
          >
            Bite my shiny metal retry!
          </button>
        </div>

        <TransitionGroup 
          name="list" 
          tag="ul" 
          class="space-y-4"
          v-else
        >
          <li 
            v-for="(entry, index) in leaderboard" 
            :key="entry.wallet_id"
            class="bg-gradient-to-r from-cyan-400 to-blue-500 rounded-lg p-6 flex items-center justify-between transition duration-300 ease-in-out hover:shadow-xl transform hover:-translate-y-1 hover:scale-105"
          >
            <div class="flex items-center space-x-4">
              <span class="text-3xl font-bold text-white bg-purple-600 rounded-full w-12 h-12 flex items-center justify-center shadow-lg">{{ index + 1 }}</span>
              <div>
                <h2 class="text-2xl font-semibold text-white">{{ getFuturamaName(entry.name) }}</h2>
                <a :href="entry.ens_address" target="_blank" rel="noopener noreferrer" class="text-yellow-300 hover:underline">
                  {{ formatENS(entry.ens_address) }}
                </a>
              </div>
            </div>
            <div class="flex items-center space-x-4">
              <span class="text-xl font-medium text-white">{{ formatBalance(entry.balance) }} NixonBucks</span>
              <span class="text-3xl animate-bounce">{{ getRandomEmoji() }}</span>
            </div>
          </li>
        </TransitionGroup>
      </div>
    </div>
  </div>
</template>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.8s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(50px) scale(0.9);
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.bg-gradient-to-br {
  background-size: 200% 200%;
  animation: gradient 15s ease infinite;
}
</style>