<script setup>
import { defineProps, defineEmits, toRefs, ref, watchEffect, onMounted } from 'vue'

let props = defineProps({
  endpoint: String,
  search: Boolean,
  sorting: Boolean,
  pagination: Boolean,
  columns: Array
})

let { endpoint, search, sorting, pagination, columns } = toRefs(props)

let emit = defineEmits(['newEmployeeAdded'])


let ascending = ref(true)
let searchTerm = ref('')
let data = ref([])
let dataFiltered = ref([])
let dataPaginated = ref([])
let pageIndex = ref(0)
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

let mapColumns = () => {
  data.value = data.value.map(entity => ({
    'name': entity.name,
    'email': entity.email.toLowerCase(),
    'company.name': entity.company.name, 
    'address.city': entity.address.city,
    'website': entity.website.toLowerCase(),
  }))
}

let searchInTable = () => {
  dataFiltered.value = []
  data.value.forEach(entity => {
    if (
      entity['name'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['email'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['company.name'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['address.city'].toLowerCase().includes(searchTerm.value.toLowerCase()) || 
      entity['website'].toLowerCase().includes(searchTerm.value.toLowerCase())
    ) {
      dataFiltered.value.push(entity)
    }
  })
  emit('change', dataFiltered.value)
}

let paginate = (e) => {
  // if (pagination) {
  //   let perPage = 3
  //   let pages = []
  //   for (let i = 0; i < dataFiltered.value.length; i += perPage) {
  //     let page = dataFiltered.value.slice(i, i + perPage);
  //     pages.push(page)
  //   }

  //   if (e.deltaY >= 0 && pageIndex.value >= 0 && pageIndex.value < pages.length - 1) pageIndex.value += 1
  //   if (e.deltaY < 0 && pageIndex.value > 0 && pageIndex.value <= pages.length - 1) pageIndex.value -= 1

  //   dataPaginated.value = pages[pageIndex.value] || pages[0]
  //   console.log(dataPaginated.value)
  // } else {
  //   dataPaginated.value = dataFiltered.value
  // }
}
    
watchEffect(async () => {
  await fetchData(endpoint.value)
  await mapColumns()
  await searchInTable()
  // await paginate()
})

let handleSort = (column) => {
  ascending.value = !ascending.value
  if (ascending.value === true) {
    dataFiltered.value.sort((a,b) => (a[column] > b[column]) ? 1 : ((b[column] > a[column]) ? -1 : 0))
  }
  if (ascending.value === false) {
    dataFiltered.value.sort((a,b) => (b[column] > a[column]) ? 1 : ((a[column] > b[column]) ? -1 : 0))
  }
}

onMounted(() => paginate())

</script>

<template>
  <section class="table" v-on:wheel="paginate" v-on:mouseenter.once="paginate">
    <div class="search" v-if="search">
      <input type="text" v-model="searchTerm" v-on:input="searchInTable">
    </div>
    <div class="grid" v-if="sorting">
      <button v-for="column in columns" :key="column" v-on:click="handleSort(column)">{{column}}</button>
    </div>
    <div class="grid" v-else>
      <button v-for="column in columns" :key="column">{{column}}</button>
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
  .table {
    min-height: 100vh;
  }
    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr)
    }
</style>