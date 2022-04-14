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
let error = ref(null)
let data = ref([])

let fetchData = async url => {
  try {
		let response = await fetch(url);
		if (!response.ok) error.value = 'Something went wrong'
		data.value = await response.json()
  } catch (err) {
    console.error(err.message)
  }
}

let mapColumns = () => {
  data.value = data.value.map(entity => ({
    'name': entity.name,
    'email': entity.email.toLowerCase(),
    'company.name': entity.company.name, 
    'address.city': entity.address.city,
    'website': entity.website.toLowerCase(),
  }))
}

watchEffect(async () => {
  await fetchData(endpoint.value)
  await mapColumns()
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
    <div class="grid">
      <button v-for="column in columns" :key="column" v-on:click="handleSort(column)">{{column}}</button>
    </div>
    <div class="grid" v-for="item in data" :key="item">
      <div v-for="column in columns" :key="column">
        <div v-if="column === 'email'"><a :href="item[column]">{{item[column]}}</a></div>
        <div v-else>{{item[column]}}</div>
      </div>
      <br>
    </div>
  </section>
</template>

<style>
  .grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr)
  }
</style>