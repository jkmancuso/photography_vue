
<template>
<div class="container-header">
    <div class="new-record"><a href="/#data-entry">Main<br>Screen</a></div>
    <div class="delete-record"><a href="/#delete-records">Delete<br>Records</a></div>
    <div class="shipping-labels"><a href="/#shipping-labels">Shipping<br>Labels</a></div>
    <div class="spreadsheet-count"><a href="/#ss">Spreadsheet<br>Count</a></div>
    <div class="create-job"><a href="/#create-job">Create<br>Job</a></div>
    <div class="modify-items"><a href="/#modify-items">Modify<br>Items</a></div>
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
.delete-record{ background-color: purple; }
.shipping-labels{ background-color: #00FF00; }
.spreadsheet-count{ background-color: #3399FF; }
.create-job{ background-color: #E0E0E0; }
.modify-items{ background-color: #F0F0F0; }
.instrument-count{ background-color: #F0F0F0; }
.delete-job{ background-color: #F0F0F0; }
</style>