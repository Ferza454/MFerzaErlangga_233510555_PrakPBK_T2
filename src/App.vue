<script setup>
import { ref, computed, onMounted } from 'vue'

// Key untuk localStorage
const STORAGE_KEY = 'vue-todo-app'

// Data reactive
const appTitle = ref('Manajemen Tugas Vue 3')
const newTask = ref({
  name: '',
  priority: 'medium',
  completed: false
})
const tasks = ref([])
const editingTaskId = ref(null)
const showCompleted = ref(true)
const nextId = ref(1)

// Load data dari localStorage saat komponen dimount
onMounted(() => {
  const savedTasks = localStorage.getItem(STORAGE_KEY)
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks)
    // Cari ID tertinggi untuk nextId
    if (tasks.value.length > 0) {
      nextId.value = Math.max(...tasks.value.map(task => task.id)) + 1
    }
  }
})

// Simpan data ke localStorage setiap kali tasks berubah
const saveToLocalStorage = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks.value))
}

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
    name: newTask.value.name.trim(),
    priority: newTask.value.priority,
    completed: false,
    createdAt: new Date().toISOString()
  })
  
  // Reset form
  newTask.value.name = ''
  newTask.value.priority = 'medium'
  
  saveToLocalStorage()
}

const deleteTask = (id) => {
  tasks.value = tasks.value.filter(task => task.id !== id)
  saveToLocalStorage()
}

const startEdit = (id) => {
  editingTaskId.value = id
}

const saveEdit = () => {
  editingTaskId.value = null
  saveToLocalStorage()
}

const cancelEdit = () => {
  editingTaskId.value = null
}

const updateTaskStatus = (task) => {
  console.log(`Status tugas "${task.name}" diubah menjadi ${task.completed ? 'selesai' : 'belum selesai'}`)
  saveToLocalStorage()
}

const clearCompleted = () => {
  tasks.value = tasks.value.filter(task => !task.completed)
  saveToLocalStorage()
}

const clearAllTasks = () => {
  if (confirm('Apakah Anda yakin ingin menghapus semua tugas?')) {
    tasks.value = []
    nextId.value = 1
    saveToLocalStorage()
  }
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
        <div>
          <h2>Daftar Tugas ({{ incompleteTasks.length }} belum selesai)</h2>
          <div class="task-meta">
            Total: {{ tasks.length }} tugas | 
            Selesai: {{ tasks.length - incompleteTasks.length }}
          </div>
        </div>
        <div class="controls-right">
          <label class="toggle-completed">
            <input type="checkbox" v-model="showCompleted">
            <span>Tampilkan tugas selesai</span>
          </label>
          <button @click="clearCompleted" class="btn btn-clear">
            Hapus Tugas Selesai
          </button>
          <button @click="clearAllTasks" class="btn btn-clear-all">
            Hapus Semua
          </button>
        </div>
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
              <div class="task-details">
                <span class="task-priority">{{ 
                  { high: 'Tinggi', medium: 'Sedang', low: 'Rendah' }[task.priority] 
                }}</span>
                <span class="task-date">
                  Dibuat: {{ new Date(task.createdAt).toLocaleDateString() }}
                </span>
              </div>
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
.task-meta {
  font-size: 0.875rem;
  color: var(--text-light);
  margin-top: 0.25rem;
}

.controls-right {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: flex-end;
}

.task-details {
  display: flex;
  gap: 1rem;
  font-size: 0.875rem;
  color: var(--text-light);
}

.task-date {
  font-style: italic;
}

.btn-clear {
  background-color: var(--warning);
  color: white;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn-clear-all {
  background-color: var(--danger);
  color: white;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn-clear:hover, .btn-clear-all:hover {
  opacity: 0.9;
}
</style>