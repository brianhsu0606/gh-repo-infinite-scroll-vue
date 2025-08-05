<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

interface Repo {
  id: number
  name: string
  description: string
  html_url: string
}

const user = 'vuejs'
const repoList = ref<Repo[]>([])

const fetchRepo = async () => {
  try {
    const res = await axios.get(`https://api.github.com/users/${user}/repos?per_page=10&page=1`)
    console.log(res.data)
    repoList.value = res.data
  } catch (error) {
    console.error('Error fetching repositories:', error)
    return []
  }
}

onMounted(() => {
  fetchRepo()
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
  <div
    v-for="repo in repoList"
    :key="repo.id"
    class="border p-4 mb-4 w-[70%] rounded-xl shadow bg-blue-300"
  >
    <h3 class="font-bold">Repo 名稱：{{ repo.name }}</h3>
    <p>Repo 描述：{{ repo.description }}</p>
    <p>Repo URL：{{ repo.html_url }}</p>
  </div>
</template>

<style scoped lang="scss"></style>
