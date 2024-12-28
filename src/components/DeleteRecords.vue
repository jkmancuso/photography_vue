<script setup>
import { ref } from 'vue'

const props = defineProps(['standardHeaders','APIBaseUrl'])

const Jobs = ref([])
const JobId = ref()
const Orders = ref([])
const gotJobIdFromSession= ref(false)
const JobName = ref()

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

  await fetch(props.APIBaseUrl + 'jobs', 
   {
    method:"GET",
    headers: props.standardHeaders
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)

    
}

async function getOrdersFromAPI(){ 
  await fetch(props.APIBaseUrl + 'jobs/' + JobId.value +'/orders', {
    method:"GET",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => Orders.value=data)

    if (Orders.value.length>0){
      JobName.value=Orders.value[0].job_name
    }
    

}


async function recreateOrderFromAPI(orderid, rec_num, jobid){
  await deleteOrderFromAPI(orderid)
  await createNewOrderFromApi(rec_num, jobid)  
  
  getOrdersFromAPI()
}

async function createNewOrderFromApi(rec_num,jobid){

  await fetch(props.APIBaseUrl + 'orders', {
    method:"POST",
    body: JSON.stringify({job_id: jobid,record_num: Number(rec_num)}),
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {console.log(data)})
}

function deleteOrderFromAPI(orderid){
  console.log("DELETE ORDER: "+ Date.now())
  fetch(props.APIBaseUrl + 'orders/' + orderid, {
    method:"DELETE",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {console.log(data)})
}

const sleep = milliseconds => {
            return new Promise(resolve => setTimeout(resolve, milliseconds));
        };

function deleteJobFromAPI(jobid){
  
  console.log("DELETE JOB: "+ Date.now())
  fetch(props.APIBaseUrl + 'jobs/' + jobid, {
    method:"DELETE",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {console.log(data)})
}

function initiateDeleteJob(){
  let confirmed = confirm("This will delete ALL orders in this job and then the job")
  
  if(!confirmed){
    return
  }

  let reversed=JSON.parse(JSON.stringify(Orders.value))
  reversed.reverse()
  for(let i in reversed){
    deleteOrderFromAPI(reversed[i].id)
    Orders.value.pop()
  }

  sleep(5000).then(() => {deleteJobFromAPI(JobId.value)});
  
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
 <div class="container">
    <div></div>
    <div>Job</div>
    <div>Order Number</div>
    <div>First Name</div>
    <div>Last Name</div>
    <div>City</div>
    <div>State</div>
 </div>
 <br>
<div class="container" v-for="order in Orders">
    <div><button @click="recreateOrderFromAPI(`${order.id}`,`${order.record_num}`,`${order.job_id}`)">Delete</button></div>
    <div>{{ order.job_name || "-"}}</div>
    <div>{{ order.record_num }}</div>
    <div>{{ order.fname || "-"}}</div>
    <div>{{ order.lname || "-"}}</div>
    <div>{{ order.city || "-"}}</div>
    <div>{{ order.state || "-"}}</div>
</div>
<br>
<br>
<div class="delete-job"><button @click="initiateDeleteJob()">Delete Job {{ JobName }}</button></div>
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
  grid-template-columns: 75px 75px 75px 75px 75px 75px 75px;
  width: 50%;
  padding: 2px;
}

.delete-job{
  width: 25%;
  border: red;
  border-style: solid;
  border-width: 1px;
  padding: 10px;
  text-align: center;
  
}

.container:hover{
background: gray; /* make this whatever you want */
}
</style>