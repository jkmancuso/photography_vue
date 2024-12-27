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

const NewGroup=ref()
const NewWoodwind =ref()
const NewBrass =ref()
const NewStrings =ref()
const NewPercussion =ref()
const NewVoice =ref()

let defaultHeaders = {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
    'X-Session-id': sessionStorage.getItem("session-id")
}

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
      headers: defaultHeaders
    })
      .then( response => response.json())
      .then(data=> obj.value = data)  
}

async function fetchInstrumentsFromApi(){
  await fetchFromAPI("instruments", Instruments)

  /*this is needed after you pull the instruments again after updating
  if you dont do this, it will APPEND to the exiting arrays below
  which will show incorrect results */
  resetInstruments()
  
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

function resetInstruments(){
  WoodwindInstruments.value=[]
  BrassInstruments.value=[]
  StringsInstruments.value=[]
  PercussionInstruments.value=[]
  VoiceInstruments.value=[]
  
  WoodwindInputRef.value=[]
  BrassInputRef.value=[]
  PercussionInputRef.value=[]
  StringsInputRef.value=[]
  VoiceInputRef.value=[]
}

async function updateInstrument(instrumentid, name){

    await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/instruments/' +
    instrumentid, {
    method: "PATCH",
    body: JSON.stringify({instrument_name: name}),
    headers: defaultHeaders
  })
    .then(response => response.json())
    .then(data => {console.log(data)})

    fetchInstrumentsFromApi()
}

async function updateGroup(groupid, name){

  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/groups/' +
    groupid, {
      method: "PATCH",
      body: JSON.stringify({group_name: name}),
      headers: defaultHeaders
      })
    .then(response => response.json())
    .then(data => {console.log(data)})
}

async function AddInstrument(instrumentname, instrumentsection){

await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/instruments',{
    method: "POST",
    body: JSON.stringify({instrument_name: instrumentname, section: instrumentsection}),
    headers: defaultHeaders
    })
  .then(response => response.json())
  .then(data => {console.log(data)})

  await fetchInstrumentsFromApi()
  resetAddFields()
}

async function AddGroup(){

await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/groups',{
    method: "POST",
    body: JSON.stringify({group_name: NewGroup.value}),
    headers: defaultHeaders
    })
  .then(response => response.json())
  .then(data => {console.log(data)})

  await fetchGroupsFromApi()
  resetAddFields()
}

function resetAddFields(){
  NewGroup.value=''
  NewWoodwind.value=''
  NewBrass.value=''
  NewPercussion.value=''
  NewStrings.value=''
  NewVoice.value=''

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
<br><button @click="AddGroup()">Add</button><input type="text" v-model="NewGroup"><br>
<br>
<div class="small-bold-italics">Woodwind</div>
<div class="container-modify">
    <div v-for="(instrument,i) in WoodwindInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${WoodwindInputRef[i]}`)">Update</button>
            <input type="text" v-model="WoodwindInputRef[i]"></div>
    </div>
</div>
<br><button @click="AddInstrument(`${NewWoodwind}`,'woodwind')">Add</button><input type="text" v-model="NewWoodwind"><br>
<br>
<div class="small-bold-italics">Brass</div>
<div class="container-modify">
    <div v-for="(instrument,i) in BrassInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${BrassInputRef[i]}`)">Update</button>
            <input type="text" v-model="BrassInputRef[i]"></div>
    </div>
</div>
<br><button @click="AddInstrument(`${NewBrass}`,'brass')">Add</button><input type="text" v-model="NewBrass"><br>
<br>
<div class="small-bold-italics">Percussion</div>
<div class="container-modify">
    <div v-for="(instrument,i) in PercussionInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${PercussionInputRef[i]}`)">Update</button>
            <input type="text" v-model="PercussionInputRef[i]"></div>
    </div>
</div>
<br><button @click="AddInstrument(`${NewPercussion}`,'percussion')">Add</button><input type="text" v-model="NewPercussion"><br>
<br>
<div class="small-bold-italics">Strings</div>
<div class="container-modify">
    <div v-for="(instrument,i) in StringsInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${StringsInputRef[i]}`)">Update</button>
            <input type="text" v-model="StringsInputRef[i]"></div>
    </div>
</div>
<br><button @click="AddInstrument(`${NewStrings}`,'strings')">Add</button><input type="text" v-model="NewStrings"><br>
<br>
<div class="small-bold-italics">Voice</div>
<div class="container-modify">
    <div v-for="(instrument,i) in VoiceInstruments">
        <div><button @click="updateInstrument(`${instrument.id}`,`${VoiceInputRef[i]}`)">Update</button>
            <input type="text" v-model="VoiceInputRef[i]"></div>
    </div>
</div>
<br><button @click="AddInstrument(`${NewVoice}`,'voice')">Add</button><input type="text" v-model="NewVoice">
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