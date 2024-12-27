<script setup>
import { ref } from 'vue'

const Jobs = ref([])
const JobId = ref()
const JobName =ref()
const Orders = ref([])
const gotJobIdFromSession= ref(false)

let GroupMap = new Map()
let JobNameMap = new Map()

fetchJobsFromAPI()  
   

async function fetchJobsFromAPI(){

  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs', 
   {
    method:"GET",
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)

    for (let i in Jobs.value){
        JobNameMap.set(Jobs.value[i].id,Jobs.value[i].job_name)
    }
    
}

async function getOrdersFromAPI(){ 
  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs/' + 
    JobId.value +'/orders', {
    method:"GET",
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
  })
    .then(response => response.json())
    .then(data => {Orders.value=data;return Orders;})

    for (let i in Orders.value){
        console.log(Orders.value[i].group)
        if(Orders.value[i].group !=null){
            GroupMap.set(Orders.value[i].group,Number(Orders.value[i].group_quantity) +1 || 1)
        }
        
    }

    JobName.value=JobNameMap.get(JobId.value)

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
<div class="small-bold-underline">Job: {{ JobName }}</div>
<br>
<div class="small-bold">Groups</div>
<div v-for="group,i in GroupMap">{{ GroupMap[i]}}</div>
</template>

<style>

td,body { font-family: "Verdana";font-size: 8pt }

.small-bold-underline{
    text-decoration-line: underline;
    font-weight: bold;
    font-size: small;

}

.small-bold{
    font-weight: bold;
    font-size: small;

}
</style>