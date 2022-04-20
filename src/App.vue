<script setup>
import { ref } from 'vue'
import Table from './components/Table.vue'

let endpoint = ref('https://jsonplaceholder.typicode.com/users')
let search = ref(true)
let sorting = ref(true)
let pagination = ref(true)
let columns = ref(['name', 'email', 'company.name', 'address.city', 'website'])

let emitChange = (e) => console.log(e)
</script>

<template>
  <Table v-on:change="emitChange"
    :endpoint="endpoint" 
    :search="search" 
    :sorting="sorting" 
    :pagination="pagination" 
    :columns="columns" 
  />
</template>