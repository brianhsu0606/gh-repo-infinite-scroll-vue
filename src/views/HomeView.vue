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

const inputUser = ref<string>('')

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
      observer?.disconnect()
    }
  } catch (error) {
    console.error('Error fetching repositories:', error)
  } finally {
    isLoading.value = false
  }
}

const loadTrigger = ref<HTMLElement | null>(null)
let observer: IntersectionObserver | null = null

const observeTrigger = () => {
  if (!loadTrigger.value) return

  observer = new IntersectionObserver((entries) => {
    const entry = entries[0]
    if (entry.isIntersecting && !isLoading.value) {
      fetchRepo()
    }
  })
  observer.observe(loadTrigger.value)
}

const submit = () => {
  if (inputUser.value.trim() === '') return
  user.value = inputUser.value.trim()
  repoList.value = []
  page.value = 1
  fetchRepo()
}

onMounted(async () => {
  await fetchRepo()
  observeTrigger()
})
</script>

<template>
  <!-- Header -->
  <header class="sticky top-0 z-10 bg-gray-200 p-4 mb-4 shadow">
    <h2 class="text-2xl font-bold text-center">獲取 GitHub Repo</h2>

    <!-- 輸入 使用者 -->
    <div class="flex items-center gap-4 mb-4">
      <p class="text-xl">GitHub 使用者</p>
      <input
        type="text"
        v-model="inputUser"
        class="border p-2 rounded-lg w-48"
        placeholder="請輸入 GitHub 使用者"
      />
      <button @click="submit" class="bg-gray-400 rounded-lg p-2 hover:bg-gray-500">輸入</button>
    </div>

    <p class="text-xl">目前使用者：{{ user }}</p>
    <p class="text-xl">目前取得的 Repo 數量: {{ repoList.length }} 筆</p>
  </header>

  <!-- Repo 卡片 -->
  <div class="flex flex-col items-center" :class="{ 'opacity-30': isLoading }">
    <div
      v-for="repo in repoList"
      :key="repo.id"
      class="border p-4 mb-4 w-[70%] rounded-xl shadow bg-blue-300 text-lg"
    >
      <h3 class="font-bold">Repo 名稱：{{ repo.name }}</h3>
      <p>Repo 描述：{{ repo.description }}</p>
      <p>
        Repo URL：
        <a :href="repo.html_url" class="text-blue-700 hover:text-red-600">
          {{ repo.html_url }}
        </a>
      </p>
    </div>
  </div>

  <!-- Loading 提示文字 -->
  <h2 v-if="isLoading" class="loading-text">載入中...</h2>

  <!-- Trigger -->
  <div ref="loadTrigger" class="h-10"></div>
</template>

<style scoped lang="scss">
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
