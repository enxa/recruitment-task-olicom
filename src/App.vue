<script setup>
import { ref } from 'vue'
import Table from './components/Table.vue'

let endpoint = ref('https://jsonplaceholder.typicode.com/users')
let search = ref(true)
let sorting = ref(false)
let pagination = ref(false)
let columns = ref(['name', 'email', 'company.name', 'address.city', 'website'])
</script>

<template>
  <Table 
    :endpoint="endpoint" 
    :search="search" 
    :sorting="sorting" 
    :pagination="pagination" 
    :columns="columns" 
  />
</template>