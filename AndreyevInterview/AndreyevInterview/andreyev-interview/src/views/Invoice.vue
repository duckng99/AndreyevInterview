<template>
  <div class="about">
    <router-link :to="{ name: 'Invoices' }">Back</router-link>

    <h2>Invoice Details</h2>

    <span>Invoice #{{$route.params.id}}</span>

    <h3>Line Items</h3>

    <table>
      <thead>
        <th>ID</th>
        <th>Description</th>
        <th>Quantity</th>
        <th>Billable</th>
        <th>Cost</th>
      </thead>
      <tbody>
        <tr v-for="item in state.lineItems" :key="item.id">
          <td>{{item.id}}</td>
          <td>{{item.description}}</td>
          <td>{{item.quantity}}</td>
          <td><input type="checkbox" :value="item" v-model="state.checked"></td>
          <td>{{item.cost}}</td>
        </tr>
        <tr>
          <td><b>Total Cost</b></td>
          <td></td>
          <td></td>
          <td></td>
          <td><b>{{ state.totalCost }}</b></td>
        </tr>
        <tr>
          <td><b>Total Billable Cost</b></td>
          <td></td>
          <td></td>
          <td></td>
          <td><b>{{  state.totalBillableCost  }}</b></td>
        </tr>
      </tbody>
    </table>
<button @click="createReport">Submit Report</button>
    <form @submit.prevent>
      <h4>Create Line Item</h4>
      <input type="text" name="description" placeholder="Description" v-model="state.description" />
      <input type="number" name="quantity" placeholder="Quantity" v-model="state.quantity" />
      <input type="number" name="cost" placeholder="Cost" v-model="state.cost" />
      <button @click="createLineItem">Create Invoice</button>
    </form>
  </div>
</template>

<script lang="ts">
import {computed, defineComponent, onMounted, reactive} from "vue";

export default defineComponent({
  name: "Invoice",
  props: {
    id: {
      type: [String, Number],
      required: true
    }
  },
  setup(props) {
    const state = reactive({
      lineItems: [],
      description: "",
      quantity: "0",
      cost: "0",
      totalCost: 0,
      checked: [],
      totalBillableCost: computed(() => {
        var result: Number
        result = state.checked.reduce(function(sum, item: any){
          return sum + item.cost
        }, 0)
        return result
      })
    })

    function fetchLineItems() {
      fetch(`http://localhost:5000/invoices/${props.id}`, {
        method: "GET",
        headers: {
          "Content-Type": "application/json"
        },
      }).then((response) => {
        //reset the total cost
        state.totalCost = 0
        response.json().then(lineItems => {
          state.lineItems = lineItems
          //calculate the total value
          for (let i = 0; i < lineItems.length; i++){
            state.totalCost += lineItems[i].cost
          }
          })
      })
    }

    function createLineItem() {
      fetch(`http://localhost:5000/invoices/${props.id}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          description: state.description,
          quantity: Number(state.quantity),
          cost: Number(state.cost)
        })
      }).then(fetchLineItems)
    }
    
    function createReport(){
      fetch("http://localhost:5000/invoices/reports", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          id: props.id,
          description: state.description,
          totalValue: Number(state.totalCost),
          totalBillableValue: Number(state.totalBillableCost),
          totalNumberLineItems: state.lineItems.length
        })
      }).then(fetchLineItems)
      alert("A report has been submitted!")
    }


    onMounted(fetchLineItems)

    return {state, createLineItem, createReport}
  }

})
</script>