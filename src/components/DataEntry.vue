<script setup>
import { ref } from 'vue'

import states from '../states.json'

const status = ref()
const returnedJSON = ref()
const jobs = ref([])
const orders = ref([])
const JobId = ref()
const RecordNum = ref(0)
const GroupPictureNumOptions = ref(["B1","C1","JE1","O1","WC1","VJ1"])
const GroupPictureNum = ref()
const Groups = ref([])
const CreatedAt = ref()
const [Year,Fname,Lname,PhoneNum,Address,City,State,Zip,Amount] =[ref(),ref(),ref(),ref(),ref(),ref(),ref(),ref(),ref()]
const [Group,GroupQuantity,CheckNum,PaymentMethod,NumberPictures] = [ref(),ref(),ref(),ref(),ref()]

// on fresh page load
if (!jobs.value.length){
  fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs')
    .then( response => response.json())
    .then(data=> jobs.value = data)
}    

//important function- it's called when user selects the job, it pulls all the orders
//treat the orders ref like a ro cache and call this func when the DB is updated
async function getOrdersFromDB(){ 
  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs/' + 
    JobId.value +'/orders')
    .then(response => response.json())
    .then(data => {orders.value=data;return orders;})
    .then(orders => RecordNum.value=orders.value.length)
  
  getJobInfo()

  if (RecordNum.value >0) {
    fillInForm()      
  } else{
    RecordNum.value = 1
  }

}

async function updateStateFromZipCode(){
  fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/zipcodes/' + 
      Zip.value)
      .then(response => response.json())
      .then(data => {State.value=data.state;City.value=data.city})
}

function fillInForm(){
  let order = orders.value[RecordNum.value-1]
  console.log(order)
  //left side are the v-model variable names
  //right side are the api json field names
  Fname.value=order.fname
  Lname.value=order.lname
  Address.value=order.address
  City.value=order.city
  State.value=order.state
  Zip.value=order.zip
  PhoneNum.value=order.phone
  Group.value=order.group
  GroupQuantity.value=order.group_quantity
  GroupPictureNum.value=order.group_picture_num
  CheckNum.value=order.check_num
  Amount.value=order.value
  PaymentMethod.value=order.payment_method
  
  CreatedAt.value=new Date(order.CreatedAt).toDateString()
  console.log(CreatedAt.value)
}

function newRecord(){
  resetForm()
  RecordNum.value=orders.value.length + 1
  orders.value.push({"record_num":RecordNum.value})
  
}

function resetForm(){
  Fname.value=''
  Lname.value=''
  Address.value=''
  City.value=''
  Zip.value=''
  PhoneNum.value=''
  GroupQuantity.value=''
  Group.value=''
  GroupPictureNum.value=''
  PaymentMethod.value=''
  CheckNum.value=''
  Amount.value=''
}

function getJobInfo(){

  for (let num in jobs.value){
    if (jobs.value[num].id == JobId.value){
      Year.value = jobs.value[num].job_year
      Groups.value= jobs.value[num].job_groups
      return
    }
  }
}
async function postOrder(){

  let json = {
        fname: Fname.value,
        lname: Lname.value,
        job_id: JobId.value,
        record_num: Number(RecordNum.value),
        address: Address.value,
        city: City.value,
        state: State.value,
        phone_num: PhoneNum.value,
        zip: Zip.value,
        group: Group.value,
        group_quantity: Number(GroupQuantity.value),
        group_picture_num: GroupPictureNum.value,
        check_num: Number(CheckNum.value),
        group_quantity:Number(GroupQuantity.value),
        amount: Number(Amount.value),
        payment_method: PaymentMethod.value
    }

  await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/orders', {
    method: "post",
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
      },
      body: JSON.stringify(json)})
      .then( response => response.json())
      .then(json => returnedJSON.value = json)
    
    orders.value.push(returnedJSON.value)
    //update the local order var instead of going to eventually consistent DynamoDB
    //GSI doesnt support consistent read (ie "read after write")
    //otherwise we would have to call getOrdersFromDB with a gross 2s sleep timer added

}
</script>

<template>
<br>
<div class="main-data-entry">Main Data Entry Screen</div>
<br>
<div class="success" v-if="status == 200">Order saved!</div>
<div class="error" v-else-if="status >= 400">Record was not saved! {{ returnedJSON }}</div>
<br>
<div class="container-main">
    <div>Job Name</div>
    <div>
        <select class="purple" v-model="JobId" @change="getOrdersFromDB">
        <option
            v-for="job in jobs"
            v-bind:value="job.id" > {{  job.job_name }}</option>
        </select>
    </div>
    <div><button v-if="orders.length>0" @click="newRecord()">NEXT RECORD</button></div>
    <div></div>

    <div>Year</div>
    <div><input class="purple" type="text" size='4' readonly v-model="Year"></div>
    
    <div v-if="orders.length > 0">Record # 
        <select v-model="RecordNum" @change="fillInForm">
            <option 
            v-for="order in orders"
            v-bind:value="order.record_num" 
            :selected="order.record_num === RecordNum">{{ order.record_num }}</option>
        </select> of {{ orders.length }}
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
    <div><input type="text" v-model="Lname"><button>Find</button></div>
    <div></div>

    <div>Street Address</div>
    <div><input type="text" v-model="Address"></div>
    <div>City</div>
    <div><input class="purple" type="text" v-model="City"> State</div>
    <div>
        <select class="purple" v-model="State">
            <option
            v-for="state in states"
            v-bind:value="state.key"
            :selected="state.key === State">{{ state.val }}</option>
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
          <option v-for="group in Groups"
          v-bind:value="group"
          :selected="group === Group">{{ group }}</option>
          </select>
        </div>
        <div>
          <select v-model="GroupPictureNum">
          <option v-for="picturenum in GroupPictureNumOptions"
          v-bind:value="picturenum"
          :selected="picturenum === GroupPictureNum"> {{ picturenum }}</option>
          </select>
        </div>
    </div>

    <div class="red">NO NOT MAKE AN ENTRY HERE<br>UNLESS STUDENT IS ORDERING GROUP</div>
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

  <div><input type="number" min="0" max="99"></div>
  <div><select></select></div>
  <div><select></select></div>
  <div><select></select></div>

  <div><input type="number" min="0" max="99"></div>
  <div><select></select></div>
  <div><select></select></div>
  <div><select></select></div>

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

  <div><input type="number" min="0" max="99"></div>
  <div><select></select></div>
  <div><select></select></div>
  <div><select></select></div>

  <div><input type="number" min="0" max="99"></div>
  <div><select></select></div>
  <div><select></select></div>
  <div><select></select></div>

  <div></div>
  <div class="section-voice">Voice Section</div>
  <div></div>

  <div class="section-accounting">
    Accounting Information<br><br>
    <div class="container-accounting">
      <div>Payment Method</div>
      <div><input type="text"  size="5" maxlength="5" v-model="PaymentMethod"></div>
      <div>Check Num</div>
      <div><input type="number"  min="0" max="99" v-model="CheckNum"></div>

      <div>Total Paid</div>
      <div><input type="number"  min="0" max="99" v-model="Amount"></div>
      <div></div>
      <div></div>

    </div>
    
  </div>
  
  <div>Qty</div>
  <div>Instrument</div>
  <div>Position</div>
  <div>Picture #</div>


  <div><input type="text" size="2" maxlength="2"></div>
  <div><select></select></div>
  <div><select></select></div>
  <div><select></select></div>

</div>

<br>
<div class="container-end">
  <div>Date Created On</div>
  <div><input type="text" class="purple" readonly v-model="CreatedAt"></div>
  <div>Number of pictures in this order</div>
  <div><input type="text" class="purple" size="2" maxlength="2" v-model="NumberPictures"></div>
  <div><button @click="postOrder">Update/Add</button></div>
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
    width: 300px;
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