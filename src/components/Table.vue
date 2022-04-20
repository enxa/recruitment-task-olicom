<script setup>
import { defineProps, toRefs, ref, watchEffect } from 'vue'

let props = defineProps({
  endpoint: String,
  search: Boolean,
  sortings: Boolean,
  pagination: Boolean,
  columns: Array
})

let { endpoint, search, sortings, pagination, columns } = toRefs(props)

let ascending = ref(true)
let searchTerm = ref('')
let data = ref([])
let dataFiltered = ref([])
let error = ref(null)

let fetchData = async url => {
  try {
		let response = await fetch(url);
		if (!response.ok) error.value = 'Something went wrong'
		data.value = await response.json()
  } catch (err) {
    error.value = err.message
  }
}

let mapColumns = async () => {
  data.value = await data.value.map(entity => ({
    'name': entity.name,
    'email': entity.email.toLowerCase(),
    'company.name': entity.company.name, 
    'address.city': entity.address.city,
    'website': entity.website.toLowerCase(),
  }))
}

let searchInTable = async () => {
  dataFiltered.value = []
  data.value.forEach(entity => {
    if (
      entity['name'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['email'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['company.name'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['address.city'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['website'].toLowerCase().includes(searchTerm.value.toLowerCase())
    ) dataFiltered.value.push(entity)
  })
}

watchEffect(async () => {
  await fetchData(endpoint.value)
  await mapColumns()
  await searchInTable()
})

let handleSort = (column) => {
  ascending.value = !ascending.value
  if (ascending.value === true) {
    data.value.sort((a,b) => (a[column] > b[column]) ? 1 : ((b[column] > a[column]) ? -1 : 0))
  }
  if (ascending.value === false) {
    data.value.sort((a,b) => (b[column] > a[column]) ? 1 : ((a[column] > b[column]) ? -1 : 0))
  }
}
</script>

<template>
  <section class="table">
    <div class="search" v-if="search">
      <input type="text" v-model="searchTerm" v-on:input="searchInTable">
    </div>
    <div class="grid">
      <button v-for="column in columns" :key="column" v-on:click="handleSort(column)">{{column}}</button>
    </div>
    <div class="grid" v-for="item in dataFiltered" :key="item">
      <div v-for="column in columns" :key="column">
        <div v-if="column === 'email'"><a :href="item[column]">{{item[column]}}</a></div>
        <div v-else>{{item[column]}}</div>
      </div>
      <br>
    </div>
    <div class="error" v-if="error">{{error}}</div>
  </section>
</template>

<style>
  .grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr)
  }
</style>