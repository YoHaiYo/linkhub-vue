<template>
  <h1>LinkHub</h1>
  <nav>
    <RouterLink to="/">Home</RouterLink>
    <RouterLink to="/about">About</RouterLink>
  </nav>
  <div>
    <RouterView />
    <hr>
    <p>title / url / description</p>
    <ul>
      <li v-for="url in urls" :key="url.id">
        <input type="text" v-model="url.title">
        <input type="text" v-model="url.url">
        <input type="text" v-model="url.description">
        <button @click="updateURL(url.id, url.title, url.url, url.description)">Save</button>
        <button @click="deleteURL(url.id)">Delete</button>
      </li>
    </ul>
    <input type="text" v-model="newTitle">
    <input type="text" v-model="newUrl">
    <input type="text" v-model="newDescription">
    <button @click="addURL">Add url</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from './config/dbconfig/supabaseClient.js'
import { RouterLink, RouterView } from 'vue-router'
// import { getURL, addURL, updateURL, deleteURL  } from './service/crud.js';

const urls = ref([])
const newTitle = ref('')
const newUrl = ref('')
const newDescription = ref('')

// 컴포넌트가 마운트될 때 데이터를 가져오는 함수
async function getURL() {
  const { data } = await supabase.from('linkhub_test').select()
  if (data) {
    urls.value = data.map(url => ({
      ...url,
      title: url.title,
      url: url.url,
      description: url.description,
    }))
  }
}

// 컴포넌트가 마운트될 때 데이터를 가져옴
onMounted(() => {
  getURL() // getURL 함수를 사용하여 데이터를 가져옵니다.
})
// 새로운 URL을 추가하는 함수
async function addURL() {
  if (newTitle.value.trim() !== '' && newUrl.value.trim() !== '' && newDescription.value.trim() !== '') {
    const { data, error } = await supabase.from('linkhub_test').insert([{ title: newTitle.value, url: newUrl.value, description: newDescription.value }])
        console.log("data1", data)
    if (error) {
      console.error('Error adding url:', error.message)
    } else {
      if (data && data.length > 0) {
        // const newURL = { ...data[0], tempTitle: data[0].title, tempUrl: data[0].url, tempDescription: data[0].description }
        // urls.value.push(newURL) // 새로운 URL을 배열에 추가
        newTitle.value = ''
        newUrl.value = ''
        newDescription.value = ''
        console.log("data2", data)
      }
    }
  }
  getURL() // 추가 후 다시 읽어서 화면 그리기
}

// URL을 업데이트하는 함수
async function updateURL(id, newTitle, newUrl, newDescription) {
  const { data, error } = await supabase.from('linkhub_test').update({ title: newTitle, url: newUrl, description: newDescription }).eq('id', id)
  if (error) {
    console.error('Error updating url:', error.message)
  } else {
    const index = urls.value.findIndex(url => url.id === id)
    if (index !== -1) {
      urls.value[index].title = newTitle
      urls.value[index].url = newUrl
      urls.value[index].description = newDescription
      urls.value[index].tempTitle = newTitle
      urls.value[index].tempUrl = newUrl
      urls.value[index].tempDescription = newDescription
    }
  }
}

// URL을 삭제하는 함수
async function deleteURL(id) {
  const { error } = await supabase.from('linkhub_test').delete().eq('id', id)
  if (error) {
    console.error('Error deleting url:', error.message)
  } else {
    urls.value = urls.value.filter(url => url.id !== id)
  }
}

</script>

<style scoped>
nav a {
  margin: .5rem;
}
</style>
