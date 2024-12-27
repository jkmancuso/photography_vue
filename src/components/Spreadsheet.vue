<script setup>
import { ref } from 'vue'

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

  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs', 
   {
    method:"GET",
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)
    
}

async function getOrdersFromAPI(){ 
  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs/' + 
    JobId.value +'/orders', {
    method:"GET",
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
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

<table border="1"><thead>
    <tr>
        <td>First Name</td>
        <td>Last Name</td>
        <td></td>
        <td>Group</td>
        <td></td>
        <td>Woodwind</td>
        <td></td>
        <td>Brass</td>
        <td></td>
        <td>Percussion</td>
        <td></td>
        <td>Strings</td>
        <td></td>
        <td>Voice</td>
    </tr></thead>
     <tr v-for="order in Orders">
        <td>{{ order.fname}}</td> 
        <td>{{ order.lname}}</td>
        <td>{{ order.group_quantity}}</td>
        <td>{{ order.group}}</td>
        <td>{{ order.section.name=="woodwind" ? order.section.quantity : ''}}</td>
        <td>{{ order.section.name=="woodwind" ? order.section.instrument : ''}}</td>
        <td>{{ order.section.name=="brass" ? order.section.quantity : ''}}</td>
        <td>{{ order.section.name=="brass" ? order.section.instrument : ''}}</td>
        <td>{{ order.section.name=="percussion" ? order.section.quantity : ''}}</td>
        <td>{{ order.section.name=="percussion" ? order.section.instrument : ''}}</td>
        <td>{{ order.section.name=="strings" ? order.section.quantity : ''}}</td>
        <td>{{ order.section.name=="strings" ? order.section.instrument : ''}}</td>
        <td>{{ order.section.name=="voice" ? order.section.quantity : ''}}</td>
        <td>{{ order.section.name=="voice" ? order.section.instrument : ''}}</td>
     </tr>
</table>
</template>

<style>

td,body { font-family: "Verdana";font-size: 8pt }

.container-labels > div {
  margin: 10px;
  padding: 20px;
  font-size: 30px;
}

thead{
    font-weight: bold;
}
.container-labels {
    
    display: grid;
    grid-template-columns: auto auto auto;
  width: 75%;
  padding: 2px;
}

</style>