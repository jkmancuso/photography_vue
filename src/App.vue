<script setup>
import { ref, computed } from 'vue'
import Topnav from './components/Header.vue'
import DataEntry from './components/DataEntry.vue'
import LoginPage from './components/LoginPage.vue'
import NotFound from './components/NotFound.vue'

const routes = {
  'data-entry': DataEntry,
  'login-page': LoginPage
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})

const loggedIn =ref()
loggedIn.value =sessionStorage.getItem("session-id")

const currentView = computed(() => 
  {return loggedIn ? routes[currentPath.value.slice(1) || '/'] : routes['login-page']}
)

</script>

<template>
  <header>
      <Topnav />
  </header>
  <component :is="currentView" />

</template>

<style scoped>

</style>
