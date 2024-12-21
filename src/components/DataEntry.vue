<script setup>
import { ref } from 'vue'

const status = ref()
const returnedJSON = ref()
const jobs = ref([])
const orders = ref([])
const selectedJobId = ref()
const selectedRecordNum = ref(1)
const fname = ref("")

// on fresh page load
if (!jobs.value.length){
    fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs')
    .then( response => response.json())
    .then(data=> {jobs.value = data})

}    

if (!orders.value.length && selectedJobId.value){
    getOrdersFromDB()
}

//treat the orders ref like a ro cache and call this func when the DB is updated
async function getOrdersFromDB(){ 
    fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/jobs/' + 
        selectedJobId.value +'/orders')
    .then( response => response.json())
    .then(data=> {console.log(data);orders.value=data})
}

function fillInForm(){
    let order = orders.value[selectedRecordNum.value-1]
    fname.value=order.fname 
}

function checkPostStatus(response){

    status.value = response.status
    if ( status.value == "200") {
        document.getElementById("MainForm").reset();
    }

    returnedJSON.value = response.json()
    return returnedJSON.value
}


async function postOrder(){

    let NextRecordNum = Number(document.getElementById('record_num').value) + 1
    let json = {
            fname: fname.value,
            lname: document.getElementById('lname').value,
            job_id: document.getElementById('job_id').value,
            record_num: NextRecordNum,
            address: document.getElementById('address').value,
            city: document.getElementById('city').value,
            state: document.getElementById('state').value,
            phone_num: document.getElementById('phone_num').value,
            check_num: Number(document.getElementById('check_num').value),
            amount: Number(document.getElementById('amount').value)
        }

    await fetch('https://ygaqa1m2xf.execute-api.us-east-2.amazonaws.com/v1/orders', {
        method: "post",
        headers: {
         'Accept': 'application/json',
         'Content-Type': 'application/json'
        },
        body: JSON.stringify(json)

    }).then( response => checkPostStatus(response))
    .then(data=> console.log(data))
    .then(orders.value.push(json))
    .then(selectedRecordNum.value=NextRecordNum) 
    //update the local order var instead of going to eventually consistent DynamoDB
    //GSI doesnt support consistent read (ie "read after write")
    //otherwise we would have to call getOrdersFromDB with a gross 2s sleep timer added
}
</script>

<template>
<br>
<div class="main-data-entry">Main Data Entry Screen</div>
<br>
{{ orders }}
<div class="success" v-if="status == 200">Order saved!</div>
<div class="error" v-else-if="status >= 400">Record was not saved! {{ returnedJSON }}</div>
<br>
<div class="container-main">
    <div>Job Name</div>
    <div>
        <select class="purple" id="job_id" v-model="selectedJobId" @change="getOrdersFromDB">
        <option
            v-for="job in jobs"
            v-bind:value="job.id" > {{  job.job_name }}</option>
        </select>
    </div>
    <div></div>
    <div></div>

    <div>Year</div>
    <div><input class="purple" type="text" size='4' maxlength='4' id="year"></div>
    <div>Record # 
        <select id="record_num" v-model="selectedRecordNum" @change="fillInForm">
            <option 
            v-for="order in orders"
            v-bind:value="order.record_num" >{{ order.record_num }}</option>
        </select> of {{ orders.length }}</div>
    <div></div>
</div>
<form id="MainForm" @submit.prevent>
<br>
<div class="small-bold-italics">Student Information</div>
<br>
<div class="container-student">
    <div>First Name</div>
    <div><input type="text" id="fname" v-model="fname"></div>
    <div>Last Name</div>
    <div><input type="text" id="lname"><button>Find</button></div>
    <div></div>

    <div>Street Address</div>
    <div><input type="text" id="address"></div>
    <div>City</div>
    <div><input class="purple" type="text" id="city"> State</div>
    <div>
        <select class="purple" id="state">
            <option value='AL'>Alabama</option>
<option value='AK'>Alaska</option>
<option value='AZ'>Arizona</option>
<option value='AR'>Arkansas</option>
<option value='CA'>California</option>
<option value='CO'>Colorado</option>
<option value='CT'>Connecticut</option>
<option value='DE'>Delaware</option>
<option value='DC'>Dist. of Columbia</option>
<option value='FL'>Florida</option>
<option value='GA'>Georgia</option>
<option value='HI'>Hawaii</option>
<option value='ID'>Idaho</option>
<option value='IL'>Illinois</option>
<option value='IN'>Indiana</option>
<option value='IA'>Iowa</option>
<option value='KS'>Kansas</option>
<option value='KY'>Kentucky</option>
<option value='LA'>Louisiana</option>
<option value='ME'>Maine</option>
<option value='MD'>Maryland</option>
<option value='MA'>Massachusetts</option>
<option value='MI'>Michigan</option>
<option value='MN'>Minnesota</option>
<option value='MS'>Mississippi</option>
<option value='MO'>Missouri</option>
<option value='MT'>Montana</option>
<option value='NE'>Nebraska</option>
<option value='NV'>Nevada</option>
<option value='NH'>New Hampshire</option>
<option value='NJ'>New Jersey</option>
<option value='NM'>New Mexico</option>
<option value='NY'>New York</option>
<option value='NC'>North Carolina</option>
<option value='ND'>North Dakota</option>
<option value='OH'>Ohio</option>
<option value='OK'>Oklahoma</option>
<option value='OR'>Oregon</option>
<option value='PW'>Palau</option>
<option value='PA'>Pennsylvania</option>
<option value='RI'>Rhode Island</option>
<option value='SC'>South Carolina</option>
<option value='SD'>South Dakota</option>
<option value='TN'>Tennessee</option>
<option value='TX'>Texas</option>
<option value='UT'>Utah</option>
<option value='VT'>Vermont</option>
<option value='VI'>Virgin Islands</option>
<option value='VA'>Virginia</option>
<option value='WA'>Washington</option>
<option value='WV'>West Virginia</option>
<option value='WI'>Wisconsin</option>
<option value='WY'>Wyoming</option>
        </select>
    </div>

    <div>Zip</div>
    <div><input type="text" size='5' maxlength='5' id="zip"><button>Find</button></div>
    <div>Phone Number</div>
    <div><input type="text" id="phone_num"></div>
    <div></div>
</div>

<br>
<div class="group-photo">
    <div class="small-bold-italics">Group Photo Order Information</div>

    <div class="container-group">
        <div>Qty</div>
        <div>Group</div>
        <div>Picture #</div>

        <div><input type="number" min="0" max="99"></div>
        <div><select><option>test</option></select></div>
        <div><select><option>test</option></select></div>
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
      <div><input type="text"  size="5" maxlength="5" id="payment_method"></div>
      <div>Check Num</div>
      <div><input type="number"  min="0" max="99" id="check_num"></div>

      <div>Total Paid</div>
      <div><input type="number"  min="0" max="99" id="amount"></div>
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
  <div><input type="text" class="purple"></div>
  <div>Number of pictures in this order</div>
  <div><input type="text" class="purple" size="2" maxlength="2"></div>
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