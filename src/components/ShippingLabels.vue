<script setup>
import { ref } from 'vue'

const props = defineProps(['standardHeaders','APIBaseUrl'])

const Jobs = ref([])
const JobId = ref()
const Orders = ref([])
const gotJobIdFromSession= ref(false)


if (sessionStorage.getItem("last-selected-job-id")){
  gotJobIdFromSession.value=true
  JobId.value=sessionStorage.getItem("last-selected-job-id")
}

if (!gotJobIdFromSession.value){
  fetchJobsFromAPI()  
} else{ //you already have the JobId so just grab its orders
  getOrdersFromAPI()
}   

async function fetchJobsFromAPI(){

  await fetch(props.APIBaseUrl+'jobs', 
   {
    method:"GET",
    headers: props.standardHeaders
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)
    
}

async function getOrdersFromAPI(){ 
  await fetch(props.APIBaseUrl+'jobs/' + JobId.value +'/orders', {
    method:"GET",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {Orders.value=data;return Orders;})
}

</script>

<template>

<br>
<div v-if="!gotJobIdFromSession">Job Name: 
        <select class="purple" v-model="JobId" @change="getOrdersFromAPI">
            <option v-for="job in Jobs" :value="job.id" > {{  job.job_name }}</option>
        </select>
</div>

<br>

<div class="container-labels">
     <div v-for="order in Orders">
        {{ order.fname}} {{ order.lname}}<br>
        {{ order.address}}<br>
        {{ order.city}}, {{ order.state}} {{ order.zip}}
    </div>
</div>
</template>

<style>

body{
    font-family: "Verdana";
    font-size: 12pt
}
.container-labels > div {
  margin: 10px;
  padding: 20px;
  font-size: 30px;
}

.container-labels {
    
    display: grid;
    grid-template-columns: auto auto auto;
  width: 75%;
  padding: 2px;
}

</style>