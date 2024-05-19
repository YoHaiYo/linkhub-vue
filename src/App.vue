<script setup>
import { ref } from 'vue'
import { supabase } from './config/dbconfig/supabaseClient.js'

const countries = ref([])
const newName = ref('')

async function getCountries() {
  const { data } = await supabase.from('countries').select()
  countries.value = data
}

async function addCountry() {
  if (newName.value.trim() !== '') {
    const { data, error } = await supabase.from('countries').insert([{ name: newName.value }])
    if (error) {
      console.error('Error adding country:', error.message)
    } else {
      if (data && data.length > 0) {
        countries.value.push(data[0])
        newName.value = ''
        await getCountries() // 추가된 데이터를 다시 가져옴
      }
    }
  }
  // 화면바로반영안됨. 임시로 새로고침으로.
  window.location.reload()
}

async function updateCountry(id, newName) {
  const { data, error } = await supabase.from('countries').update({ name: newName }).eq('id', id)
  if (error) {
    console.error('Error updating country:', error.message)
  } else {
    const index = countries.value.findIndex(country => country.id === id)
    if (index !== -1) {
      countries.value[index].name = newName
    }
  }
}

async function deleteCountry(id) {
  const { error } = await supabase.from('countries').delete().eq('id', id)
  if (error) {
    console.error('Error deleting country:', error.message)
  } else {
    countries.value = countries.value.filter(country => country.id !== id)
  }
}

getCountries() // 컴포넌트가 마운트될 때 데이터를 가져옴
</script>

<template>
  <h1>LinkHub</h1>
  <div>
    <ul>
      <li v-for="country in countries" :key="country.id">
        <input type="text" v-model="country.name" @blur="updateCountry(country.id, country.name)">
        <button @click="deleteCountry(country.id)">Delete</button>
      </li>
    </ul>
    <input type="text" v-model="newName">
    <button @click="addCountry">Add Country</button>
  </div>
</template>
