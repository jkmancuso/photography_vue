<script setup>
import { ref } from 'vue'

const props = defineProps(['standardHeaders','APIBaseUrl'])

const Jobs = ref([])
const JobId = ref()
const JobName =ref()
const Orders = ref([])
const gotJobIdFromSession= ref(false)

let GroupMap = new Map()
let JobNameMap = new Map()
let InstrumentJSON= {
  "woodwind":{},
  "brass":{},
  "strings":{},
  "percussion":{},
  "voice":{}
}

if (sessionStorage.getItem("last-selected-job-id")){
  JobId.value=sessionStorage.getItem("last-selected-job-id")
  getOrdersFromAPI()
}

fetchJobsFromAPI()  

async function fetchJobsFromAPI(){

  await fetch(props.APIBaseUrl + 'jobs', 
   {
    method:"GET",
    headers: props.standardHeaders
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)

  for (let i in Jobs.value){
    JobNameMap.set(Jobs.value[i].id,Jobs.value[i].job_name)
  }
    
}

async function getOrdersFromAPI(){ 
  
  await fetch(props.APIBaseUrl+'jobs/' + JobId.value +'/orders', {
    method:"GET",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {Orders.value=data;return Orders;})

  for (let i in Orders.value){
    let order=Orders.value[i]

    if(order.group !=null){
      GroupMap.set(order.group,Number(GroupMap.get(order.group)) +1 || 1)
    }
    
    //section is woodwind, brass, etc
    let section=order.section.name

    let instrument=order.section.instrument

    if(section != null){
      let existing_quantity= InstrumentJSON[section][instrument] || 0
      InstrumentJSON[section][instrument] =existing_quantity +1 
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
<div v-for="group in GroupMap">{{ group[0]}}: {{ group[1]}}</div>
<br>
<div class="small-bold">Instruments</div>
<br>
<div v-for="quantity,instrument in InstrumentJSON.woodwind">
  <span class="small-bold">woodwind</span> {{ instrument}}: {{quantity}}</div>

<div v-for="quantity,instrument in InstrumentJSON.brass">
  <span class="small-bold">brass</span> {{ instrument}}: {{quantity}}</div>

<div v-for="quantity,instrument in InstrumentJSON.percussion">
  <span class="small-bold">percussion</span> {{ instrument}}: {{quantity}}</div>

<div v-for="quantity,instrument in InstrumentJSON.strings">
  <span class="small-bold">strings</span> {{ instrument}}: {{quantity}}</div>
        
<div v-for="quantity,instrument in InstrumentJSON.voice">
  <span class="small-bold">voice</span> {{ instrument}}: {{quantity}}</div>

</template>

<style>

body { font-family: "Verdana";font-size: 8pt }

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