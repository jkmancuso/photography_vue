
<template>
<div class="container-header">
    <div class="new-record"><a href="/#data-entry">New<br>Record</a></div>
    <div class="prev-record">Previous<br>Record</div>
    <div class="delete-record">Delete<br>Record</div>
    <div class="shipping-labels">Shipping<br>Labels</div>
    <div class="spreadsheet-count">Spreadsheet<br>Count</div>
    <div class="clear-screen">Clear/Main<br>Screen</div>
    <div class="create-job">Create<br>Job</div>
    <div class="modify-items">Modify<br>Items</div>
    <div class="instrument-count">Instrument/Group<br>Count</div>
    <div class="delete-job">Delete<br>A Job</div>
</div>
</template>

<style>
.container-header {
  display: flex;
  flex-wrap: nowrap;

}

.container-header > div {
    font-size: medium;
    width: 125px;
    padding: 5px;
    margin: 2px;
    border-color: black;
    border-style: solid;
    border-width: 1px;
}

.new-record{ background-color: #007F00; }
.prev-record{ background-color: yellow; }
.delete-record{ background-color: purple; }
.shipping-labels{ background-color: #00FF00; }
.spreadsheet-count{ background-color: #3399FF; }
.clear-screen{ background-color: #C0C0C0; }
.create-job{ background-color: #E0E0E0; }
.modify-items{ background-color: #F0F0F0; }
.instrument-count{ background-color: #F0F0F0; }
.delete-job{ background-color: #F0F0F0; }
</style>