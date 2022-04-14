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

let data = ref([])
let cols = ref([])
let entities = ref(new Set([]))
let error = ref(null)

let fetchData = async url => {
  try {
		let response = await fetch(url);
		if (!response.ok) error.value = 'Something went wrong'
		data.value = await response.json()
  } catch (err) {
    console.error(err.message)
  }
}

watchEffect(() => fetchData(endpoint.value))

watchEffect(() => {
  data.value.forEach(row => {
    let entity = []
    columns.value.forEach(col => {
      entity.push(eval(`row.${col}`))
      entities.value.add(entity)
    })
  })
})
</script>

<template>
  <section class="table">
    <div v-for="column in columns" :key="column">
      <button>{{column}}</button>
    </div>
    <div v-for="entity in entities" :key="entity" class="table">
      <br>
      <div v-for="en in entity" :key="en">
        <span>{{en}}</span>
      </div>
      <br>
    </div>
  </section>
</template>