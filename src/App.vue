<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('')
const input_content = ref('')
const input_category = ref(null)
const temperature = ref(null) // Menyimpan suhu udara

const todos_asc = computed(() => todos.value.sort((a, b) => {
    return a.createdAt - b.createdAt
}))

watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
}, {
    deep: true
})

const addTodo = () => {
    if (input_content.value.trim() === '' || input_category.value === null) {
        return
    }

    todos.value.push({
        content: input_content.value,
        category: input_category.value,
        done: false,
        editable: false,
        createdAt: new Date().getTime() // Simpan timestamp saat tugas dibuat
    })

    // Reset input fields
    input_content.value = ''
    input_category.value = null
}

const removeTodo = (todo) => {
    todos.value = todos.value.filter((t) => t !== todo)
}

// Fungsi untuk mengambil suhu dari WeatherAPI
const fetchTemperature = async () => {
    const apiKey = '2633b310f3c04698bc1191423240211' // Ganti dengan API key Anda
    const city = 'Jakarta' // Ganti dengan kota yang diinginkan
    const url = `https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${city}&aqi=no`
    
    try {
        const response = await fetch(url)
        if (!response.ok) {
            throw new Error('Respons jaringan tidak ok')
        }
        const data = await response.json()
        if (data.current && data.current.temp_c !== undefined) {
            temperature.value = data.current.temp_c // Simpan suhu dalam Celsius
        } else {
            temperature.value = 'Data tidak tersedia'
        }
    } catch (error) {
        console.error('Kesalahan mengambil suhu:', error)
        temperature.value = 'Kesalahan mengambil data'
    }
}

onMounted(() => {
    name.value = localStorage.getItem('name') || ''
    todos.value = JSON.parse(localStorage.getItem('todos')) || []
    fetchTemperature() // Panggil fungsi untuk mengambil suhu saat komponen dimuat
})
</script>

<template>
    <main class="app">
        <section class="greeting">
            <h2 class="title">
                Selamat Datang, <input type="text" id="name" placeholder="Nama di sini" v-model="name">
            </h2>
            <div class="temperature">
                Suhu: {{ temperature !== null ? temperature + ' °C' : 'Memuat...' }}
            </div>
        </section>

        <section class="create-todo">
            <h3>Buat Tugas</h3>

            <form id="new-todo-form" @submit.prevent="addTodo">
                <h4>Apa yang Anda Mau Lakukan?</h4>
                <input 
                    type="text" 
                    name="content" 
                    id="content" 
                    placeholder="contoh: buat video"
                    v-model="input_content" />
                
                <h4>Pilih kategori</h4>
                <div class="options">
                    <label>
                        <input 
                            type="radio" 
                            name="category" 
                            id="category1" 
                            value="business"
                            v-model="input_category" />
                        <span class="bubble business"></span>
                        <div>Bisnis</div>
                    </label>

                    <label>
                        <input 
                            type="radio" 
                            name="category" 
                            id="category2" 
                            value="personal"
                            v-model="input_category" />
                        <span class="bubble personal"></span>
                        <div>Pribadi</div>
                    </label>
                </div>

                <input type="submit" value="Tambah tugas" />
            </form>
        </section>

        <section class="todo-list">
            <h3>DAFTAR TUGAS</h3>
            <div class="list" id="todo-list">
                <div v-for="todo in todos_asc" :key="todo.createdAt" :class="`todo-item ${todo.done && 'done'}`">
                    <label>
                        <input type="checkbox" v-model="todo.done" />
                        <span :class="`bubble ${
                            todo.category == 'business' 
                                ? 'business' 
                                : 'personal'
                        }`"></span>
                    </label>

                    <div class="todo-content">
                        <input type="text" v-model="todo.content" />
                    </div>

                    <div class="date">
                        <small>Dibuat pada: {{ new Date(todo.createdAt).toLocaleString() }}</small>
                    </div>

                    <div class="actions">
                        <button class="delete" @click="removeTodo(todo)">Hapus</button>
                    </div>
                </div>
            </div>
        </section>
    </main>
</template>

<style scoped>
.temperature {
    position: absolute; /* Mengatur posisi suhu di pojok kanan atas */
    right: 20px;
    top: 20px;
    background: rgba(255, 255, 255, 0.8); /* Latar belakang semi-transparan */
    padding: 0.5rem;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}
</style>