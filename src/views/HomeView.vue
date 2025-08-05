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

const fetchRepo = async () => {
  try {
    const res = await axios.get('https://api.github.com/users/vuejs/repos?per_page=10&page=1')
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
