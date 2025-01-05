<script setup>
import { ref } from 'vue'

import states from '../states.json'
import woodwind_pictures from '../woodwind_pictures.json'
import brass_pictures from '../brass_pictures.json'
import percussion_pictures from '../percussion_pictures.json'
import strings_pictures from '../strings_pictures.json'
import voice_pictures from '../voice_pictures.json'

const props = defineProps(['standardHeaders','APIBaseUrl'])

const Status = ref()
const ReturnedJSON = ref()
const Jobs = ref([])
const Order = ref()
const SelectedInstrument = ref()
const SelectedPictureNum = ref()

const [Orders,DisplayOrders] = [ref([]),ref([])]
const [RecordNum] = [ref(0)]
const JobId = ref()
const OrderId =ref()

const GroupPictureNumOptions = ["B1","C1","JE1","O1","WC1","VJ1"]
const GroupPictureNum = ref()
const Groups = ref([])

let LastNames={}

const [Year,Fname,Lname,PhoneNum,Address,City,State,Zip,Amount] =[ref(),ref(),ref(),ref(),ref(),ref(),ref(),ref(),ref()]
const [Group,GroupQuantity,CheckNum,PaymentMethod] = [ref(),ref(),ref(),ref()]
const PrevName = ref()

const Instruments = ref([])

const Positions = ["","1st","2nd","3rd"]
const [WoodwindInstruments,BrassInstruments,PercussionInstruments,StringsInstruments,VoiceInstruments]=  [ref([]),ref([]),ref([]),ref([]),ref([])]

const CreatedAt = ref()

const SectionNames=["woodwind","brass","percussion","strings","voice"]

const SectionMap = ref({
  woodwind: {quantity: "",instrument:"",position:"",picture_num:""},
  brass:{quantity: "",instrument:"",position:"",picture_num:""},
  percussion:{quantity: "",instrument:"",position:"",picture_num:""},
  strings:{quantity: "",instrument:"",position:"",picture_num:""},
  voice:{quantity: "",instrument:"",position:"",picture_num:""}
})
let JobMap = new Map()

// on fresh page load
if (!Jobs.value.length){
  fetchJobsFromAPI()  
}    

// on fresh page load
if (!Instruments.value.length){
  fetchInstrumentsFromAPI()
}    

async function fetchJobsFromAPI(){
  await fetch(props.APIBaseUrl+'jobs', 
   {
    method:"GET",
    headers: props.standardHeaders
  })
    .then( response => response.json())
    .then(data=> Jobs.value = data)

    //grab the job name
    for (let i in Jobs.value){
      JobMap.set(Jobs.value[i].id,Jobs.value[i].job_name)
    }

}

async function fetchInstrumentsFromAPI(){
  await fetch(props.APIBaseUrl + 'instruments', {
    method:"GET",
    headers: props.standardHeaders
  })
    .then( response => response.json())
    .then(data=> Instruments.value = data)
    
    loadInstruments()
}

function loadInstruments(){
  
  for (let num in Instruments.value){

    let instrument = Instruments.value[num]
    
    if (instrument.section == "woodwind"){
      WoodwindInstruments.value.push(instrument.instrument_name)
    } else if (instrument.section == "brass") {
      BrassInstruments.value.push(instrument.instrument_name)
    } else if (instrument.section == "percussion") {
      PercussionInstruments.value.push(instrument.instrument_name)
    } else if (instrument.section == "strings") {
      StringsInstruments.value.push(instrument.instrument_name)
    } else if (instrument.section == "voice") {
      VoiceInstruments.value.push(instrument.instrument_name)
    }
  }

  WoodwindInstruments.value.sort()
  BrassInstruments.value.sort()
  PercussionInstruments.value.sort()
  StringsInstruments.value.sort()
  VoiceInstruments.value.sort()

}
//important function- it's called when user selects the job, it pulls all the orders
//treat the orders ref like a ro cache and call this func when the DB is updated
async function getOrdersFromAPI(){ 
  await fetch(props.APIBaseUrl+'jobs/' + JobId.value +'/orders', {
    method:"GET",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {Orders.value=data;return Orders;})
    .then(Orders => RecordNum.value=Orders.value.length)
  
  for (let order of Orders.value){
    LastNames[order.lname]=order.record_num
  }  
  
  DisplayOrders.value=JSON.parse(JSON.stringify(Orders.value))
  getYearInfo()
  getGroupsFromAPI()

  if (RecordNum.value >0) {
    fillInForm()      
  } else{
    RecordNum.value = 1
  }

  sessionStorage.setItem("last-selected-job-id",JobId.value)

}

function FindByLastName(lname){
  RecordNum.value=LastNames[lname]
  fillInForm()
}

async function getGroupsFromAPI(){
  fetch(props.APIBaseUrl+'groups', {
    method:"GET",
    headers: props.standardHeaders
  })
      .then(response => response.json())
      .then(data => Groups.value=data)
}


async function updateStateFromZipCode(){
  fetch(props.APIBaseUrl + 'zipcodes/' + Zip.value, {
    method:"GET",
    headers: props.standardHeaders
  })
    .then(response => response.json())
    .then(data => {State.value=data.state;City.value=data.city})
}

function fillInForm(){
  resetForm()

  // Orders array starts at 0 but RecordNum starts at index 1
  // so subtract 1 from RecordNum
  Order.value = Orders.value[RecordNum.value-1]

  if (Order.value == null){
    //this is probably the empty record you created
    //by click NEXT RECORD
    return
  }

  //left side are the v-model variable names
  //right side are the api json field names
  OrderId.value=Order.value.id
  Fname.value=Order.value.fname
  Lname.value=Order.value.lname
  Address.value=Order.value.address
  City.value=Order.value.city
  State.value=Order.value.state
  Zip.value=Order.value.zip
  PhoneNum.value=Order.value.phone
  Group.value=Order.value.group
  GroupQuantity.value=Order.value.group_quantity
  GroupPictureNum.value=Order.value.group_picture_num
  CheckNum.value=Order.value.check_num
  Amount.value=Order.value.amount
  PaymentMethod.value=Order.value.payment_method
  
  const section = Order.value.section

  //its optional to add a section to the order
  if (section?.name != null){
    SelectedInstrument.value= Order.value.section.instrument || ""
    SelectedPictureNum.value= Order.value.section.picture_num || ""
    
    SectionMap.value[section.name].quantity=section.quantity || 0
    SectionMap.value[section.name].instrument=section.instrument || ""
    SectionMap.value[section.name].position=section.position || ""
    SectionMap.value[section.name].picture_num=section.picture_num || ""
  }
  
  CreatedAt.value=new Date(Order.value.CreatedAt).toDateString()

}

/* when you click the "NEXT RECORD" button
little bit of a hack here
So, When you click the button it adds NOT to the real Orders
but instead to a "DisplayOrders" which is what populates the 
Record # dropdown. If we didn't do this, we'd have to create 
a real, mostly blank record each time */

function newRecord(){

  //prevent the user from clicking repeatedly 
  // and getting too far ahead
  if(DisplayOrders.value.length>Orders.value.length){
    return
  }

  RecordNum.value++
  DisplayOrders.value.push({"record_num":RecordNum.value})
  resetForm()
  
  }

//document.getElementById('MainForm').reset() not working :(
function resetForm(){
  
  //this one is our hidden element used for updating DB
  OrderId.value=''
  
  Fname.value=''
  Lname.value=''
  Address.value=''
  City.value=''
  Zip.value=''
  PhoneNum.value=''
  GroupQuantity.value=''
  Group.value=''
  GroupPictureNum.value=''
  
  for (let thesection of SectionNames){
    SectionMap.value[thesection].quantity =''
    SectionMap.value[thesection].instrument =''
    SectionMap.value[thesection].position =''
    SectionMap.value[thesection].picture_num =''  
  }

  PaymentMethod.value=''
  CheckNum.value=''
  Amount.value=''

  CreatedAt.value=''
  
}

function getYearInfo(){

  for (let num in Jobs.value){
    if (Jobs.value[num].id == JobId.value){
      Year.value = Jobs.value[num].job_year
      return
    }
  }
}

async function postOrder(json){

  
  await fetch(props.APIBaseUrl + 'orders', {
    method: "POST",
    headers: props.standardHeaders,
    body: JSON.stringify(json)
    })
    .then(response => {Status.value =response.status;return response.json()})
    .then(json => ReturnedJSON.value = json)
    
  if (Status.value == 200) {
    PrevName.value = Fname.value + " " + Lname.value
    Orders.value.push(ReturnedJSON.value)
    DisplayOrders.value=JSON.parse(JSON.stringify(Orders.value)) 
    //update the local order var instead of going to eventually consistent DynamoDB
    //GSI doesnt support consistent read (ie "read after write")
    //otherwise we would have to call getOrdersFromAPI with a gross 2s sleep timer added
    //console.log("UPDATED Order count: " + Orders.value.length)
    }
    

}

async function patchOrder(json){

  await fetch(props.APIBaseUrl+'orders/'+ OrderId.value, {
    method: "PATCH",
    headers: props.standardHeaders,
    body: JSON.stringify(json)})
    .then(response => {Status.value =response.status;return response.json()})
    .then(json => ReturnedJSON.value = json)
  
  if (Status.value == 200) {
    PrevName.value = Fname.value + " " + Lname.value
    //to do fix orders-Orders.value.push(ReturnedJSON.value)
    getOrdersFromAPI()
  }
  

}

function PostOrPatchOrder(){

  let sectionjson = {}

  for (let thesection of SectionNames){
    let sectionquantity = Number(SectionMap.value[thesection].quantity)
    
    // there are 5 sections [woodwind,brass,percussion,strings,voice]
    // only add the section to the DB for the one that has a quantity assigned 
    if(sectionquantity >0){
      sectionjson.quantity= sectionquantity
      sectionjson.name = thesection
      sectionjson.instrument=SectionMap.value[thesection].instrument
      sectionjson.position=SectionMap.value[thesection].position
      sectionjson.picture_num=SectionMap.value[thesection].picture_num
    }
  }  

  let json = {
        fname: Fname.value,
        lname: Lname.value,
        job_id: JobId.value,
        job_name: JobMap.get(JobId.value),
        record_num: Number(RecordNum.value),
        address: Address.value,
        city: City.value,
        state: State.value,
        phone: PhoneNum.value,
        zip: Zip.value,
        group_quantity: Number(GroupQuantity.value),
        group: Group.value,
        group_picture_num: GroupPictureNum.value,
        section: sectionjson,
        payment_method: PaymentMethod.value,
        check_num: Number(CheckNum.value),
        amount: Number(Amount.value)
    }


  if (!OrderId.value){
    postOrder(json)
  } else{
    patchOrder(json)
  }
}
</script>

<template>
<br>
<div class="main-data-entry">Main Data Entry Screen</div>
<br> 
<div class="success" v-if="Status == 200">Order {{PrevName}} saved !</div>
<div class="error" v-else-if="Status >= 400">Record was not saved! {{ ReturnedJSON }}</div>
<br> <input type="hidden" v-model="OrderId">
<div class="container-main">
    <div>Job Name</div>
    <div>
        <select class="purple" v-model="JobId" @change="getOrdersFromAPI">
        <option v-for="job in Jobs" :value="job.id" > {{  job.job_name }}</option>
        </select>
    </div>
    <div><button v-if="Orders.length>0 && RecordNum >= Orders.length" @click="newRecord()">NEXT RECORD</button></div>
    <div></div>

    <div>Year</div>
    <div><input class="purple" type="text" size='4' readonly v-model="Year"></div>
    
    <div v-if="Orders.length > 0">Record # 
        <select v-model="RecordNum" @change="fillInForm">
            <option v-for="order in DisplayOrders" :value="order.record_num"  >{{ order.record_num }}</option>
        </select> of {{ DisplayOrders.length }}
    </div>
    <div v-else>First record</div>
    <div></div>
</div>
<form id="MainForm" @submit.prevent>
<br>
<div class="small-bold-italics">Student Information</div>
<br>
<div class="container-student">
    <div>First Name</div>
    <div><input type="text" v-model="Fname"></div>
    <div>Last Name</div>
    <div><input type="text" v-model="Lname"><button @click="FindByLastName(`${Lname}`)">Find</button></div>
    <div></div>

    <div>Street Address</div>
    <div><input type="text" v-model="Address"></div>
    <div>City</div>
    <div><input class="purple" type="text" v-model="City"> State</div>
    <div>
        <select class="purple" v-model="State">
            <option v-for="state in states" :value="state.key">{{ state.val }}</option>
        </select>
    </div>

    <div>Zip</div>
    <div>
      <input type="text" size='5' maxlength='5' v-model="Zip">
      <button @click="updateStateFromZipCode">Find</button>
    </div>
    <div>Phone Number</div>
    <div><input type="text" v-model="PhoneNum"></div>
    <div></div>
</div>

<br>
<div class="group-photo">
    <div class="small-bold-italics">Group Photo Order Information</div>

    <div class="container-group">
        <div>Qty</div>
        <div>Group</div>
        <div>Picture #</div>

        <div><input type="number" min="0" max="99" v-model="GroupQuantity"></div>
        <div>
          <select v-model="Group">
          <option v-for="group in Groups" :value="group.group_name">{{ group.group_name }}</option>
          </select>
        </div>
        <div>
          <select v-model="GroupPictureNum">
          <option v-for="picturenum in GroupPictureNumOptions" :value="picturenum"> {{ picturenum }}</option>
          </select>
        </div>
    </div>

    <div class="red">DO NOT MAKE AN ENTRY HERE<br>UNLESS STUDENT IS ORDERING GROUP</div>
</div>

<br>
<div class="small-bold-italics">Section Order Information</div>
<br>

<div class="container-section">

<div></div>
<div class="section-woodwind">Woodwind Section</div>
<div></div>

<div></div>
<div class="section-brass">Brass Section</div>
<div></div>

<div>Qty</div>
<div>Instrument</div>
<div>Position</div>
<div>Picture #</div>

<div>Qty</div>
<div>Instrument</div>
<div>Position</div>
<div>Picture #</div>

<div><input type="number" min="0" max="99" v-model='SectionMap.woodwind.quantity'></div>
<div>
  <select v-model="SectionMap.woodwind.instrument">
    <option v-for="instrument in WoodwindInstruments" :value="instrument">{{ instrument }}</option>
  </select>
  </div>
<div><select v-model="SectionMap.woodwind.position">
  <option v-for="position in Positions" :value="position"> {{ position }}</option>
</select></div>
<div>
  <select v-model="SectionMap.woodwind.picture_num">
    <option v-for="picture_num in woodwind_pictures" :value="picture_num.picture"> {{ picture_num.picture }}</option>
  </select>
</div>

<div><input type="number" min="0" max="99" v-model="SectionMap.brass.quantity"></div>
<div>
  <select v-model="SectionMap.brass.instrument">
    <option v-for="instrument in BrassInstruments" :value="instrument">{{ instrument }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.brass.position">
    <option v-for="position in Positions" :value="position"> {{ position }}</option>
  </select></div>
<div>
  <select v-model="SectionMap.brass.picture_num">
    <option v-for="picture_num in brass_pictures" :value="picture_num.picture"> {{ picture_num.picture }}</option>
  </select>
</div>

<div></div>
<div class="section-percussion">Percussion Section</div>
<div></div>

<div></div>
<div class="section-strings">Strings Section</div>
<div></div>

<div>Qty</div>
<div>Instrument</div>
<div>Position</div>
<div>Picture #</div>

<div>Qty</div>
<div>Instrument</div>
<div>Position</div>
<div>Picture #</div>

<div><input type="number" min="0" max="99" v-model="SectionMap.percussion.quantity"></div>
<div>
  <select v-model="SectionMap.percussion.instrument">
    <option v-for="instrument in PercussionInstruments" :value="instrument">{{ instrument }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.percussion.position">
    <option v-for="position in Positions" :value="position"> {{ position }}</option>
  </select></div>
<div>
  <select v-model="SectionMap.percussion.picture_num">
    <option v-for="picture_num in percussion_pictures" :value="picture_num.picture"> {{ picture_num.picture }}</option>
  </select>
</div>

<div><input type="number" min="0" max="99" v-model="SectionMap.strings.quantity"></div>
<div>
  <select v-model="SectionMap.strings.instrument">
    <option v-for="instrument in StringsInstruments" :value="instrument">{{ instrument }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.strings.position">
    <option v-for="position in Positions" :value="position"> {{ position }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.strings.picture_num">
    <option v-for="picture_num in strings_pictures" :value="picture_num.picture"> {{ picture_num.picture }}</option>
  </select>
</div>

<div></div>
<div class="section-voice">Voice Section</div>
<div></div>

<div class="section-accounting">
  Accounting Information<br><br>
  <div class="container-accounting">
    <div>Payment Method</div>
    <div><input type="text"  size="5" maxlength="10" v-model="PaymentMethod"></div>
    <div>Check Num</div>
    <div><input type="number"  min="0" max="99999" v-model="CheckNum"></div>

    <div>Total Paid</div>
    <div><input type="number"  min="0" max="999" v-model="Amount"></div>
    <div></div>
    <div></div>

  </div>
  
</div>

<div>Qty</div>
<div>Instrument</div>
<div>Position</div>
<div>Picture #</div>


<div><input type="number" min="0" max="99" v-model="SectionMap.voice.quantity"></div>
<div>
  <select v-model="SectionMap.voice.instrument">
    <option v-for="instrument in VoiceInstruments" :value="instrument">{{ instrument }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.voice.position">
    <option v-for="position in Positions" :value="position"> {{ position }}</option>
  </select>
</div>
<div>
  <select v-model="SectionMap.voice.picture">
    <option v-for="picture_num in voice_pictures" :value="picture_num.picture"> {{ picture_num.picture }}</option>
  </select>
</div>

</div>

<br>
<div class="container-end">
  <div>Date Created On</div>
  <div><input type="text" class="purple" readonly v-model="CreatedAt"></div>
  <div>Number of pictures in this order</div>
  <div><input type="text" class="purple" size="2" maxlength="2" 
  :value="(GroupQuantity || 0) +
    (SectionMap.woodwind.quantity || 0) + 
    (SectionMap.brass.quantity || 0) +
    (SectionMap.percussion.quantity || 0) +
    (SectionMap.strings.quantity || 0) +
    (SectionMap.voice.quantity || 0)"></div>
  <div><button @click="PostOrPatchOrder">Update/Add</button></div>
  <div></div>
</div>

</form>
</template>



<style>
.container-end {
  display: grid;
  grid-template-columns: auto auto;
  width: 25%;
}


.container-end > div {
    padding: 2px;
  
 }

.group-photo{
    padding: 2px;
    border-style: solid;
    border-width: 1px;
    border-color: black;
    width: 400px;
}
body{
    font-family: "Verdana";
    font-size: 8pt
}

.red{
  color: red;
}

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

.container-group {
  display: grid;
  grid-template-columns: auto auto auto;
}

.container-group > div {
    padding: 5px;
    margin: 2px;
 }

 .container-section {
  display: grid;
  grid-template-columns: auto auto auto auto auto auto auto auto;
  width: 40%;
}

.container-section > div {
    padding: 5px;
 }

 .section-woodwind{
  grid-column: 2/ span 2;
  color: #00FF00;
 }

 .section-brass{
  grid-column: 6/ span 2;
  color: #0000AA;
 }

 .section-percussion{
  grid-column: 2/ span 2;
  color: #CC0099;
 }

 .section-strings{
  grid-column: 6/ span 2;
  color: #008000;
 }

.section-voice{
  grid-column: 2/ span 2;
  color: #00CCCC;
 }


 .section-accounting{
  grid-column: 5/ span 4;
  grid-row: 7/ span 3;
  border: black;
  border-style: solid;
  border-width: 1px;
  background-color: yellow;
 }

 .container-accounting{
  display: grid;
  grid-template-columns: auto auto auto auto;
 }

 .container-accounting > div {
    padding: 2px;
    
 }

 .container-main {
  display: grid;
  grid-template-columns: auto auto auto auto;
  width: 750px;
}
.container-main > div {
  padding: 2px;
  text-align: left;
}

.container-student {
  display: grid;
  grid-template-columns: auto auto auto auto auto;
  width: 750px;
}
.container-student > div {
  padding: 2px;
  text-align: left;
}

.purple {
    background-color: #FF99FF;
}

.main-data-entry{
    font-style: italic;
    font-weight: bold;
    text-decoration-line: underline;
    font-size: large;
}

.small-bold-italics{
    font-style: italic;
    font-weight: bold;
    font-size: small;
}
</style>