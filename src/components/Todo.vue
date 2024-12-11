<script setup>
import { ref , onMounted } from 'vue';
import axios from 'axios'


const task = ref([]);
const newTask = ref('');
const editTask = ref('');
const editingIndex = ref(null);
const isModalOpen = ref(false);


onMounted (async ()=>{
    const response = await axios.get("http://localhost:8000/tasks")
    console.log(response.data)
    task.value=response.data
})


const handleSubmit = async () => {
  if (newTask.value.trim() !== '') {
    try {
      const response = await axios.post("http://localhost:8000/tasks", { title: newTask.value });
      task.value.push(response.data);
      newTask.value = '';
    } catch (error) {
      console.error("Error adding task:", error);
    }
  }
};

const handleEdit = (taskToEdit, index) => {
  editTask.value = taskToEdit;
  editingIndex.value = index;
  isModalOpen.value = true;
};

const handleUpdate = async () => {
  if (editTask.value.trim() !== '') {
    try {
      const updatedTask = { ...task.value[editingIndex.value], title: editTask.value };
      await axios.put(`http://localhost:8000/tasks/${updatedTask.id}`, updatedTask);
      task.value[editingIndex.value] = updatedTask;
      isModalOpen.value = false;
      editingIndex.value = null;
      editTask.value = '';
    } catch (error) {
      console.error("Error updating task:", error);
    }
  }
};

const handleDelete = async (t) => {
  try {
    await axios.delete(`http://localhost:8000/tasks/${t.id}`);
    task.value = task.value.filter(taskItem => taskItem.id !== t.id);
  } catch (error) {
    console.error("Error deleting task:", error);
  }
};
</script>

<template>
  <div class="flex items-center justify-center min-h-screen">
    <div class="w-full max-w-md p-6 bg-white shadow-lg rounded-lg">
      <form @submit.prevent="handleSubmit" class="mb-4 flex justify-center items-center">
        <input type="text" placeholder="Add task" id="newTask" name="newTask" v-model="newTask" class="border  p-2 rounded mr-2 w-full ">
        <button type="submit" class="bg-green-500 text-white p-2 rounded hover:bg-green-600 w-32">Add Task</button>
      </form>

      <ul class="list-none p-0">
        <li v-for="(t, index) in task" :key="index" class="mb-2 flex items-center justify-between p-2  rounded hover:bg-gray-200">
          <span>{{ t.title }}</span>
          <div class="space-x-2">
            <button @click="handleDelete(t)" class="bg-red-500 text-white p-1 rounded hover:bg-red-600">Delete</button>
            <button @click="handleEdit(t.title, index)" class="bg-green-500 text-white p-1 rounded hover:bg-green-600">Edit</button>
          </div>
        </li>
      </ul>


      <div v-if="isModalOpen" class="fixed top-0 left-0 w-full h-full bg-black bg-opacity-50 flex justify-center items-center">
        <div class="bg-white p-6 rounded-lg shadow-lg w-1/3">
          <h2 class="text-xl mb-4 text-center">Edit Task</h2>
          <input v-model="editTask" type="text" placeholder="Edit your task" class="border border-gray-300 p-2 rounded w-full mb-4">
          <div class="flex justify-center space-x-4">
            <button @click="handleUpdate" class="bg-green-500 text-white p-2 rounded hover:bg-green-600 w-32">Update Task</button>
            <button @click="isModalOpen = false" class="bg-blue-400 text-white p-2 rounded hover:bg-blue-600 w-32">Cancel</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
