<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

interface Repo {
  id: number
  name: string
  description: string
  html_url: string
}

const repoList = ref<Repo[]>([])
const user = ref<string>('vuejs')
const page = ref<number>(1)
const isLoading = ref<boolean>(false)
const isEmpty = ref<boolean>(false)

// 獲取指定使用者的 GitHub Repo 資料（每次 10 筆）
const fetchRepo = async () => {
  isLoading.value = true
  try {
    const res = await axios.get(
      `https://api.github.com/users/${user.value}/repos?per_page=10&page=${page.value}`,
    )
    if (res.data.length > 0) {
      repoList.value.push(...res.data)
      page.value++
    } else {
      resetObserver()
      isEmpty.value = true
    }
  } catch (error) {
    console.error('Error fetching repositories:', error)
  } finally {
    isLoading.value = false
  }
}

// 重置 IntersectionObserver
const resetObserver = () => {
  observer?.disconnect()
  observer = null
}

// 使用 IntersectionObserver 觀察 trigger，進行無限滾動載入
const loadTrigger = ref<HTMLElement | null>(null)
let observer: IntersectionObserver | null = null

const observeTrigger = () => {
  if (!loadTrigger.value) return

  resetObserver()

  observer = new IntersectionObserver((entries) => {
    const entry = entries[0]
    if (entry.isIntersecting && !isLoading.value) {
      fetchRepo()
    }
  })
  observer.observe(loadTrigger.value)
}

// 搜尋新的 GitHub 使用者，並重新載入其 Repo 資料
const inputUser = ref<string>('')

const submit = async () => {
  if (inputUser.value.trim() === '') return

  resetObserver()

  user.value = inputUser.value.trim()
  repoList.value = []
  page.value = 1
  isEmpty.value = false
  inputUser.value = ''

  await fetchRepo()
  observeTrigger()
}

onMounted(async () => {
  await fetchRepo()
  observeTrigger()
})
</script>

<template>
  <div class="sticky top-0 z-10">
    <!-- Header -->
    <header class="p-4 border-b border-gray-400 bg-blue-300">
      <h2 class="text-2xl font-bold text-center">獲取 GitHub Repository</h2>
    </header>

    <!-- Section -->
    <section class="repo-section">
      <!-- 目前使用者、目前 Repo 數量 -->
      <div class="text-xl">
        <p>
          GitHub 帳號：<span class="font-bold">{{ user }}</span>
        </p>
        <p>
          已讀取 Repo 數量: <span class="font-bold">{{ repoList.length }}</span> 筆
        </p>
      </div>

      <!-- 輸入框 -->
      <div class="flex flex-col items-start gap-2 sm:flex-row sm:items-center w-full sm:w-auto">
        <p class="font-bold">GitHub 帳號搜尋</p>
        <div class="flex items-center gap-2 w-full sm:w-auto">
          <input
            type="text"
            v-model="inputUser"
            class="rounded-lg px-4 py-2 text-lg w-[70%]"
            placeholder="請輸入 GitHub 帳號"
            @keyup.enter="submit"
          />
          <button
            @click="submit"
            class="rounded-lg px-4 py-2 text-lg bg-blue-400 hover:bg-blue-500 w-[30%]"
          >
            確認
          </button>
        </div>
      </div>
    </section>
  </div>

  <!-- Repo 卡片 -->
  <div class="flex flex-col items-center" :class="{ 'opacity-30': isLoading }">
    <div v-for="(repo, index) in repoList" :key="repo.id" class="repo-card">
      <p>
        <span class="font-bold">編號：</span>
        {{ index + 1 }}
      </p>
      <h3>
        <span class="font-bold">Repo 名稱：</span>
        {{ repo.name }}
      </h3>
      <p>
        <span class="font-bold">Repo 描述：</span>
        {{ repo.description }}
      </p>
      <p>
        <span class="font-bold">Repo 連結：</span>
        <a :href="repo.html_url" class="text-blue-700 hover:text-red-600">
          {{ repo.html_url }}
        </a>
      </p>
    </div>
  </div>

  <!-- Repo 取完時的提示 -->
  <h2 v-if="isEmpty" class="text-2xl font-bold text-center">此帳號的 Repo 已全部讀取！</h2>

  <!-- Loading 提示文字 -->
  <h2 v-if="isLoading" class="loading-text">載入中...</h2>

  <!-- Trigger -->
  <div ref="loadTrigger" class="h-4"></div>
</template>

<style scoped lang="scss">
.repo-section {
  @apply flex flex-col justify-between items-start gap-6 
  sm:flex-row sm:items-center sm:gap-0 
  px-8 py-4 mb-4
  text-lg sm:text-xl
  border-b border-gray-400 
  bg-blue-200;
}

.repo-card {
  @apply rounded-xl shadow-lg
  p-4 mb-4
  text-sm sm:text-xl
  w-[90%] max-w-6xl
  border border-gray-400
  bg-blue-100;
}

.loading-text {
  position: fixed;
  z-index: 10;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 2rem;
  font-weight: bold;
}
</style>
