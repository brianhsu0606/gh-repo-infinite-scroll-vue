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
  <div v-for="repo in repoList" :key="repo.id">
    <h3>{{ repo.name }}</h3>
    <p>{{ repo.description }}</p>
    <p>{{ repo.html_url }}</p>
  </div>
</template>

<style scoped lang="scss"></style>
