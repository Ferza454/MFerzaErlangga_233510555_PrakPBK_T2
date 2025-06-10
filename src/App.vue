<script setup>
import { ref, computed } from 'vue'

// Data reactive
const appTitle = ref('Manajemen Tugas Vue 3')
const newTask = ref({
  name: '',
  priority: 'medium',
  completed: false
})
const tasks = ref([
  { id: 1, name: 'Belajar Vue.js', priority: 'high', completed: false },
  { id: 2, name: 'Buat proyek kecil', priority: 'medium', completed: true },
  { id: 3, name: 'Deploy aplikasi', priority: 'low', completed: false }
])
const editingTaskId = ref(null)
const showCompleted = ref(true)
const nextId = ref(4)

// Computed properties
const incompleteTasks = computed(() => {
  return tasks.value.filter(task => !task.completed)
})

const filteredTasks = computed(() => {
  return showCompleted.value 
    ? tasks.value 
    : tasks.value.filter(task => !task.completed)
})

// Methods
const addTask = () => {
  if (newTask.value.name.trim() === '') return
  
  tasks.value.push({
    id: nextId.value++,
    name: newTask.value.name,
    priority: newTask.value.priority,
    completed: false
  })
  
  // Reset form
  newTask.value.name = ''
  newTask.value.priority = 'medium'
}

const deleteTask = (id) => {
  tasks.value = tasks.value.filter(task => task.id !== id)
}

const startEdit = (id) => {
  editingTaskId.value = id
}

const saveEdit = () => {
  editingTaskId.value = null
}

const cancelEdit = () => {
  editingTaskId.value = null
}

const updateTaskStatus = (task) => {
  console.log(`Status tugas "${task.name}" diubah menjadi ${task.completed ? 'selesai' : 'belum selesai'}`)
}
</script>

<template>
  <div class="container">
    <h1>{{ appTitle }}</h1>
    
    <!-- Form Bindings -->
    <form @submit.prevent="addTask" class="task-form">
      <div class="form-group">
        <label for="taskName">Nama Tugas:</label>
        <input 
          type="text" 
          id="taskName" 
          v-model="newTask.name" 
          placeholder="Masukkan nama tugas"
        >
      </div>
      <div class="form-group">
        <label for="taskPriority">Prioritas:</label>
        <select id="taskPriority" v-model="newTask.priority">
          <option value="high">Tinggi</option>
          <option value="medium">Sedang</option>
          <option value="low">Rendah</option>
        </select>
      </div>
      <button type="submit" class="btn">Tambah Tugas</button>
    </form>
    
    <!-- Conditional Rendering -->
    <div v-if="tasks.length === 0" class="empty-state">
      <p>Tidak ada tugas. Tambahkan tugas pertama Anda!</p>
    </div>
    
    <div v-else>
      <div class="task-controls">
        <h2>Daftar Tugas ({{ incompleteTasks.length }} belum selesai)</h2>
        <label class="toggle-completed">
          <input type="checkbox" v-model="showCompleted">
          <span>Tampilkan tugas selesai</span>
        </label>
      </div>
      
      <!-- Attribute Bindings -->
      <ul class="task-list">
        <li 
          v-for="task in filteredTasks" 
          :key="task.id" 
          class="task-item"
          :class="{
            'completed': task.completed,
            [`priority-${task.priority}`]: true
          }"
        >
          <div class="task-content">
            <input 
              type="checkbox" 
              v-model="task.completed"
              @change="updateTaskStatus(task)"
              class="task-checkbox"
            >
            
            <!-- Conditional Rendering untuk mode edit/tampil -->
            <div v-if="editingTaskId !== task.id" class="task-info">
              <span class="task-name">{{ task.name }}</span>
              <span class="task-priority">{{ 
                { high: 'Tinggi', medium: 'Sedang', low: 'Rendah' }[task.priority] 
              }}</span>
            </div>
            
            <div v-else class="task-edit">
              <input 
                type="text" 
                v-model="task.name"
                @keyup.enter="saveEdit"
                class="edit-input"
              >
              <select v-model="task.priority" class="edit-select">
                <option value="high">Tinggi</option>
                <option value="medium">Sedang</option>
                <option value="low">Rendah</option>
              </select>
            </div>
          </div>
          
          <div class="task-actions">
            <!-- Event Listeners -->
            <button 
              v-if="editingTaskId !== task.id"
              @click="startEdit(task.id)" 
              class="btn btn-edit"
            >
              Edit
            </button>
            <button 
              v-else
              @click="saveEdit" 
              class="btn btn-save"
            >
              Simpan
            </button>
            <button 
              v-if="editingTaskId === task.id"
              @click="cancelEdit" 
              class="btn btn-cancel"
            >
              Batal
            </button>
            <button 
              @click="deleteTask(task.id)" 
              class="btn btn-delete"
            >
              Hapus
            </button>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
/* CSS akan ditempatkan di file terpisah */
</style>