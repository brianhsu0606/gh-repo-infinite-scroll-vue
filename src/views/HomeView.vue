<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const repoList = ref([])

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
  <div>{{ repoList }}</div>
</template>

<style scoped lang="scss"></style>
