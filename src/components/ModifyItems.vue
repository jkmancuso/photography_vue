<script setup>
import { ref } from 'vue'

const WoodwindInputRef = ref([])
const PercussionInputRef = ref([])
const BrassInputRef = ref([])
const StringsInputRef = ref([])
const VoiceInputRef = ref([])

const Instruments =ref([])

const WoodwindInstruments = ref([])
const BrassInstruments = ref([])
const PercussionInstruments = ref([])
const StringsInstruments = ref([])
const VoiceInstruments = ref([])
const GroupInputRef = ref([])
const Groups =ref([])


// on fresh page load
if (!Instruments.value.length){
  fetchInstrumentsFromApi() 
}    

// on fresh page load
if (!Groups.value.length){
  fetchGroupsFromApi() 
}    

async function fetchGroupsFromApi(){
  await fetchFromAPI("groups", Groups)

  for (let i in  Groups.value){
          GroupInputRef.value[i] = Groups.value[i].group_name
      }
}

async function fetchFromAPI(type, obj) {
  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/'+type, 
     {
      method:"GET",
      headers: { 'x-session-id': sessionStorage.getItem("session-id") }
    })
      .then( response => response.json())
      .then(data=> obj.value = data)  
}

async function fetchInstrumentsFromApi(){
  await fetchFromAPI("instruments", Instruments)
  
  for (let i in  Instruments.value){
    if (Instruments.value[i].section == "woodwind"){
        WoodwindInstruments.value.push(Instruments.value[i])
        WoodwindInputRef.value.push(Instruments.value[i].instrument_name)
    } else if (Instruments.value[i].section == "brass") {
        BrassInstruments.value.push(Instruments.value[i])
        BrassInputRef.value.push(Instruments.value[i].instrument_name)
    } else if (Instruments.value[i].section == "percussion") {
        PercussionInstruments.value.push(Instruments.value[i])
        PercussionInputRef.value.push(Instruments.value[i].instrument_name)
    } else if (Instruments.value[i].section == "strings") {
        StringsInstruments.value.push(Instruments.value[i])
        StringsInputRef.value.push(Instruments.value[i].instrument_name)
    } else if (Instruments.value[i].section == "voice") {
        VoiceInstruments.value.push(Instruments.value[i])
        VoiceInputRef.value.push(Instruments.value[i].instrument_name)
    }      
  }
}

async function updateInstrument(instrumentid, name){

    await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/instruments/' +
    instrumentid, {
    method: "PATCH",
    body: JSON.stringify({instrument_name: name}),
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      'X-Session-id': sessionStorage.getItem("session-id")
      }
  })
    .then(response => response.json())
    .then(data => {console.log(data)})
}

async function updateGroup(groupid, name){

await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/groups/' +
groupid, {
method: "PATCH",
body: JSON.stringify({group_name: name}),
headers: {
  'Accept': 'application/json',
  'Content-Type': 'application/json',
  'X-Session-id': sessionStorage.getItem("session-id")
  }
})
.then(response => response.json())
.then(data => {console.log(data)})
}


</script>

<template>
<div class="small-bold-italics">Groups</div>
<div class="container-modify">
    <div v-for="(group,i) in Groups">
        <div><button @click="updateGroup(`${group.id}`,`${GroupInputRef[i]}`)">Update</button>
            <input type="text" v-model="GroupInputRef[i]"></div>
    </div>
</div>

<br>
<div class="small-bold-italics">Woodwind</div>
<div class="container-modify">
    <div v-for="(instrument,i) in WoodwindInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${WoodwindInputRef[i]}`)">Update</button>
            <input type="text" v-model="WoodwindInputRef[i]"></div>
    </div>
</div>

<br>
<div class="small-bold-italics">Brass</div>
<div class="container-modify">
    <div v-for="(instrument,i) in BrassInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${BrassInputRef[i]}`)">Update</button>
            <input type="text" v-model="BrassInputRef[i]"></div>
    </div>
</div>

<br>
<div class="small-bold-italics">Percussion</div>
<div class="container-modify">
    <div v-for="(instrument,i) in PercussionInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${PercussionInputRef[i]}`)">Update</button>
            <input type="text" v-model="PercussionInputRef[i]"></div>
    </div>
</div>

<br>
<div class="small-bold-italics">Strings</div>
<div class="container-modify">
    <div v-for="(instrument,i) in StringsInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${StringsInputRef[i]}`)">Update</button>
            <input type="text" v-model="StringsInputRef[i]"></div>
    </div>
</div>

<br>
<div class="small-bold-italics">Voice</div>
<div class="container-modify">
    <div v-for="(instrument,i) in VoiceInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${VoiceInputRef[i]}`)">Update</button>
            <input type="text" v-model="VoiceInputRef[i]"></div>
    </div>
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

.container-modify {
  display: grid;
  grid-template-columns: auto auto auto auto;
  width: 90%;

}

.container-modify > div {
    padding: 5px;
 }

 .small-bold-italics{
    font-style: italic;
    font-weight: bold;
    font-size: small;
}
</style>