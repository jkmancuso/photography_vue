<script setup>
import { ref, computed } from 'vue'
import Topnav from './components/Header.vue'
import DataEntry from './components/DataEntry.vue'
import DeleteRecords from './components/DeleteRecords.vue'
import ShippingLabels from './components/ShippingLabels.vue'
import Spreadsheet from './components/Spreadsheet.vue'
import CreateJob from './components/CreateJob.vue'
import ModifyItems from './components/ModifyItems.vue'
import Count from './components/Count.vue'
import NotFound from './components/NotFound.vue'


const Status = ref()
const ReturnedJSON = ref()
const Email =ref()
const Password=ref()
const LoginErr = ref()

let APIBaseUrl="https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/"
let standardHeaders = { 
   'Accept': 'application/json',
   'Content-Type': 'application/json',
   'x-session-id': sessionStorage.getItem("session-id")
  }

// by default on page load you are not logged in 
const loggedIn =ref(false)
// but if you have a valid session id in your browser local storage then you are good
if (sessionStorage.getItem("session-id")){
  if (sessionStorage.getItem("expire-at") > Math.round(new Date().getTime() / 1000) ){
    loggedIn.value=true
  }
}

async function postAuth(json){

  json={
      email: Email.value,
      password:Password.value
  }

  await fetch(APIBaseUrl +'auth', {
    method: "POST",
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
      },
      body: JSON.stringify(json)})
      .then(response => {Status.value =response.status; return response.json()})
      .then(json => ReturnedJSON.value = json)
    
  if (Status.value == 200) {
    handleLoginOK()  
  } else{
    handleLoginFail()
  }
  
}

function handleLoginOK(){

  sessionStorage.setItem("session-id",ReturnedJSON.value.id)
  sessionStorage.setItem("expire-at",ReturnedJSON.value.expire_at)
  standardHeaders = { 
   'Accept': 'application/json',
   'Content-Type': 'application/json',
   'x-session-id': ReturnedJSON.value.id
  }
  loggedIn.value=true
}

function handleLoginFail(){
  LoginErr.value="Unable to login: "+ ReturnedJSON.value.message
}

const routes = {
  'data-entry': DataEntry,
  '/': DataEntry,
  'delete-records': DeleteRecords,
  'shipping-labels':ShippingLabels,
  'ss': Spreadsheet,
  'create-job':CreateJob,
  'modify-items': ModifyItems,
  'count': Count

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

  <div v-if="!loggedIn" class="logincontainer">   
    <div>
    <label for="exampleInputEmail1">Email address</label><br>
    <input v-model="Email" type="email"  id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter email">
    
  </div>
  <div >
    <label for="exampleInputPassword1">Password</label><br>
    <input v-model="Password" type="password"  id="exampleInputPassword1" placeholder="Password">
  </div>
<br>
  <button @click="postAuth" type="submit" >Submit</button>
</div>
 <div class="error">{{ LoginErr }}</div>
  <component :is="currentView"  
    v-if="loggedIn" 
    :standardHeaders="standardHeaders"
    :APIBaseUrl="APIBaseUrl"
    />

</template>

<style scoped>

div{
    font-family: "Verdana";
    font-size: 12pt
}

.logincontainer {
    
    width: 50%;
    text-align: center;    
  
  }
  .error{
  color: red;
  font-weight: bold;
  font-size: large;
  text-align: center;
}
</style>