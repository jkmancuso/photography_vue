<script setup>
import { ref } from 'vue'

const props = defineProps(['standardHeaders','APIBaseUrl'])

const Name = ref()
const Year= ref()
const ReturnedJSON = ref()
const Status =ref()

async function postNewJob(){  
    await fetch(props.APIBaseUrl + 'jobs', {
    method:"POST",
    body: JSON.stringify({job_name: Name.value,job_year: Number(Year.value)}),
    headers: props.standardHeaders
  })
    .then(response => {Status.value=response.status; return response.json()})
    .then(data => {ReturnedJSON.value=data;console.log(data)})

}

</script>

<template>

<br>
Job Name: <input v-model="Name"> Year: <input v-model="Year"> 
<button @click="postNewJob">Add Job</button>
<div class="success" v-if="Status == 200">Job saved !</div>
<div class="error" v-else-if="Status >= 400">Job not saved! {{ ReturnedJSON.message }}</div>

</template>

<style>
.error{
  color: red;
  font-weight: bold;
  font-size: large
}

.success{
  color: blue;
  font-weight: bold;
  font-size: large
}
</style>