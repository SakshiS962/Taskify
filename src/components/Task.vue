<script setup>
import { onMounted, ref } from 'vue'
import Localbase from 'localbase'

let db = new Localbase('taskify')

const showModal = ref(false)

const tasks = ref([])

const isCompleted = ref([])

onMounted(() => {
  tasks.value.map((t) => {
    isCompleted.value.push(t.isCompleted)
  })
})

// const handleCheckbox = (e) => {
//   const uTasks = tasks.value.map((t) => {
//     if (t.id === parseInt(e.target.value)) {
//       return { ...t, isCompleted: !t.isCompleted }
//     } else {
//       return t
//     }
//   })
//   tasks.value = uTasks
// }

const handleCheckbox = async (e) => {
  try {
    const uTasks = tasks.value.find((t) => {
      return t.id === parseInt(e.target.value)
    })
    console.log(uTasks)
    await db
      .collection('tasks')
      .doc({ id: parseInt(e.target.value) })
      .update({ isCompleted: !uTasks.isCompleted })

    getTasks()
  } catch (e) {
    console.log(e)
  }
}
const handleAddTask = () => {
  selectedTask.value = null
  showModal.value = true
  newTask.value = ''
}
const newTask = ref('')
const selectedTask = ref()

// const addNewTask = () => {
//   const task = {
//     id: Date.now(),
//     name: newTask.value,
//     isCompleted: false
//   }
//   showModal.value = false
// }

const addNewTask = async () => {
  try {
    const task = {
      id: Date.now(),
      name: newTask.value,
      isCompleted: false
    }
    await db.collection('tasks').add(task)
    getTasks()
    showModal.value = false
  } catch (e) {
    console.log(e)
  }
}
const getTasks = async () => {
  try {
    const data = await db.collection('tasks').get()
    tasks.value = data.sort((a, b) => b.id - a.id)
    //tasks.value = data
  } catch (e) {
    console.log(e)
  }
}

onMounted(() => {
  getTasks()
})

const editTask = (task) => {
  selectedTask.value = task
  newTask.value = task.name
  showModal.value = true
}

// const updateTask = () => {
//   const uTasks = tasks.value.map((t) => {
//     if (t.id === selectedTask.value.id) {
//       return { ...t, name: newTask.value }
//     } else {
//       return t
//     }
//   })
//   tasks.value = uTasks
//   newTask.value = ''
//   showModal.value = false
// }

const updateTask = async () => {
  try {
    await db.collection('tasks').doc({ id: selectedTask.value.id }).update({
      name: newTask.value
    })
    getTasks()
    newTask.value = ''
    showModal.value = false
  } catch (e) {
    console.log(e)
  }
}
// const deleteTask =  (task) => {
//   const uTasks = tasks.value.filter((t) => {
//     return t.id !== task.id
//   })
//   tasks.value = uTasks
//   }

const deleteTask = async (task) => {
  try {
    await db.collection('tasks').doc({ id: task.id }).delete()
    getTasks()
  } catch (e) {
    console.log(e)
  }
}
</script>

<template>
  <header class="header">
    <div class="container">
      <h1>Taskify</h1>
      <button class="plus" @click="handleAddTask"><i class="fa-solid fa-plus"></i></button>
    </div>
  </header>
  <div class="container">
    <ul class="list">
      <li class="list-item" v-for="(task, i) in tasks" :key="task.id">
        <span :class="task.isCompleted ? 'strick' : ' '"> {{ task.name }}</span>
        <div class="icons">
          <i @click="editTask(task)" class="fa-solid fa-pen-to-square edit-icon"></i>
          <i @click="deleteTask(task)" class="fa-solid fa-trash-can delete-icon"></i>
          <input
            type="checkbox"
            :value="task.id"
            v-model="isCompleted[i]"
            @change="handleCheckbox($event)"
          />
        </div>
      </li>
    </ul>
  </div>

  <!-- dailog -->
  <div v-if="showModal" class="dailog">
    <div class="dailog-card">
      <div class="header">
        <h3>{{ selectedTask ? 'Edit' : 'Add' }} Task</h3>
        <button @click="showModal = false" class="cross-icon">
          <i class="fa-solid fa-xmark"></i>
        </button>
      </div>
      <div class="dailog-body">
        <label>Task Name:</label><br />
        <input type="text" placeholder="Enter Task Name" v-model="newTask" />
      </div>
      <div class="footer">
        <button @click="selectedTask ? updateTask() : addNewTask()" class="footer-button">
          {{ selectedTask ? 'Update' : 'Add' }} Task
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
