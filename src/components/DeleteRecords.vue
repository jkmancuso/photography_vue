<script setup>
import { ref } from 'vue'

const Jobs = ref([])
const JobId = ref()
const Orders = ref([])

// on fresh page load
if (!Jobs.value.length){
  fetchJobsFromAPI()  
}    

async function fetchJobsFromAPI(){
  fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs', 
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

async function recreateOrderFromAPI(orderid, rec_num, jobid){
  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/orders/' + 
    orderid, {
    method:"DELETE",
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
  })
    .then(response => response.json())
    .then(data => {console.log(data)})
    
    await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/orders', {
    method:"POST",
    body: JSON.stringify({job_id: jobid,record_num: Number(rec_num)}),
    headers: { 'x-session-id': sessionStorage.getItem("session-id") }
  })
    .then(response => response.json())
    .then(data => {console.log(data)})

    getOrdersFromAPI()
}


</script>

<template>

<br>
<div>Job Name: 
        <select class="purple" v-model="JobId" @change="getOrdersFromAPI">
            <option v-for="job in Jobs" :value="job.id" > {{  job.job_name }}</option>
        </select>
</div>

<br>
 <div class="container">
    <div></div>
    <div>Order Number</div>
    <div>First Name</div>
    <div>Last Name</div>
    <div>City</div>
    <div>State</div>
 </div>
 <br>
<div class="container" v-for="order in Orders">
    <div><button @click="recreateOrderFromAPI(`${order.id}`,`${order.record_num}`,`${order.job_id}`)">Delete</button></div>
    <div>{{ order.record_num }}</div>
    <div>{{ order.fname || "-"}}</div>
    <div>{{ order.lname || "-"}}</div>
    <div>{{ order.city || "-"}}</div>
    <div>{{ order.state || "-"}}</div>
</div>
</template>

<style>
.purple {
    background-color: #FF99FF;
}

body{
    font-family: "Verdana";
    font-size: 12pt
}

.container {
  display: grid;
  grid-template-columns: 75px 75px 75px 75px 75px 75px;
  width: 50%;
  padding: 2px;
}



.container:hover{
background: gray; /* make this whatever you want */
}
</style>