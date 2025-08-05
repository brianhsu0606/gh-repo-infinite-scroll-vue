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
const user: string = 'vuejs'
const page = ref<number>(1)
const isLoading = ref<boolean>(false)

const fetchRepo = async () => {
  isLoading.value = true
  try {
    const res = await axios.get(
      `https://api.github.com/users/${user}/repos?per_page=10&page=${page.value}`,
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

onMounted(() => {
  fetchRepo()
  observeTrigger()
})
</script>

<template>
  <!-- Header -->
  <header class="bg-gray-200 p-4 mb-4 shadow">
    <h2 class="text-2xl font-bold text-center">獲取 GitHub Repo</h2>
    <p class="text-xl">目標使用者：{{ user }}</p>
    <p class="text-xl">目前取得的 Repo 數量: {{ repoList.length }} 筆</p>
  </header>

  <!-- Repo 卡片 -->
  <div class="flex flex-col items-center">
    <div
      v-for="repo in repoList"
      :key="repo.id"
      class="border p-4 mb-4 w-[70%] rounded-xl shadow bg-blue-300"
    >
      <h3 class="font-bold">Repo 名稱：{{ repo.name }}</h3>
      <p>Repo 描述：{{ repo.description }}</p>
      <p>Repo URL：{{ repo.html_url }}</p>
    </div>
  </div>

  <!-- Trigger -->
  <div ref="loadTrigger" class="h-10 text-center bg-gray-200">Trigger</div>
</template>

<style scoped lang="scss"></style>
