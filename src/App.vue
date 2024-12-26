<script setup>
import { ref, computed } from 'vue'
import Topnav from './components/Header.vue'
import DataEntry from './components/DataEntry.vue'
import NotFound from './components/NotFound.vue'
const loggedIn =ref(false)


const Status = ref()
const ReturnedJSON = ref()
const ResponseHeaders= ref()
const Email =ref()
const Password=ref()

async function postAuth(json){

  json={
      email: Email.value,
      password:Password.value
  }

  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/auth', {
    method: "POST",
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
      },
      body: JSON.stringify(json)})
      .then(response => {Status.value =response.status; return response.json()})
      .then(json => ReturnedJSON.value = json)
    
  if (Status.value == 200) {
    handleOK()
    }
  

}

function handleOK(){
  console.log(Status.value)
      console.log(ReturnedJSON.value)
      console.log(ResponseHeaders.value)

      sessionStorage.setItem("session-id",ReturnedJSON.value.id)
      sessionStorage.setItem("expire-at",ReturnedJSON.value.expire_at)
      loggedIn.value=true
}

const routes = {
  'data-entry': DataEntry,
  '/': DataEntry
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})


const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/'] || NotFound
})


</script>

<template>
  <header>
      <Topnav />
  </header>

  <div v-if="!loggedIn" class="container">   
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <div>
    <label for="exampleInputEmail1">Email address</label><br>
    <input v-model="Email" type="email" class="form-control-sm" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter email">
    
  </div>
  <div class="form-group">
    <label for="exampleInputPassword1">Password</label><br>
    <input v-model="Password" type="password" class="form-control-sm" id="exampleInputPassword1" placeholder="Password">
  </div>
<br>
  <button @click="postAuth" type="submit" class="btn btn-primary">Submit</button>
</div>
 
  <component :is="currentView"  v-if="loggedIn"/>

</template>

<style scoped>
.container {
    
    width: 50%;
    text-align: center;    
  
  }
</style>
