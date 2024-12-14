<template>
  <div class="bg-gray-900 min-h-screen text-white font-mono">
    <div class="container mx-auto p-8">
      <h1 class="text-3xl font-bold mb-6">Task Management App</h1>

      <button
        class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded mb-6"
        @click="state.showAddTaskForm = !state.showAddTaskForm"
      >
        {{
          state.showAddTaskForm ? "Hide Add Task Form" : "Show Add Task Form"
        }}
      </button>

      <div v-if="state.showAddTaskForm" class="mb-6">
        <input
          type="text"
          v-model="state.newTask"
          class="bg-gray-800 border border-gray-700 rounded py-2 px-4 w-full"
          placeholder="Enter new task (at least 3 characters)"
        />
        <button
          class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded mt-2"
          @click="addTask"
        >
          Add Task
        </button>
      </div>

      <div class="mb-4">
        <button
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mr-2"
          @click="state.showCompleted = !state.showCompleted"
        >
          {{ state.showCompleted ? "Show All Tasks" : "Show Completed Tasks" }}
        </button>
      </div>

      <div v-if="filteredTasks.length === 0" class="mb-4">
        No tasks available.
      </div>

      <div
        v-for="task in filteredTasks"
        :key="task.id"
        class="flex items-center mb-4"
      >
        <input
          type="text"
          disabled
          v-model="task.taskTitle"
          :class="{
            'bg-gray-800': !task.completed,
            'bg-green-400': task.completed,
            'border-red-500 border-2': task.completed,
            'line-through': task.completed,
          }"
          class="rounded py-2 px-4 w-full mr-4"
        />
        <button
          v-if="!task.completed"
          class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mr-2"
          @click="completeTask(task.id)"
        >
          Complete
        </button>
        <button
          v-else
          class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded mr-2"
          @click="undoTask(task.id)"
        >
          Undo
        </button>
        <button
          class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
          @click="deleteTask(task.id)"
        >
          Delete
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted, watch } from "vue";

function generateId() {
  return Math.random().toString(36).substring(2, 15);
}

const state = reactive({
  showAddTaskForm: false,
  newTask: "",
  tasks: [],
  showCompleted: false,
});

onMounted(() => {
  const storedTasks = localStorage.getItem("tasks");
  if (storedTasks) {
    state.tasks = JSON.parse(storedTasks);
  }
});

watch(
  () => state.tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
  },
  { deep: true }
);

const addTask = () => {
  if (state.newTask.trim() !== "" && state.newTask.trim().length >= 3) {
    state.tasks.push({
      id: generateId(),
      taskTitle: state.newTask,
      completed: false,
    });
    state.newTask = "";
  } else {
    alert("Task name must be at least 3 characters long.");
  }
};

const completeTask = (taskId) => {
  const task = state.tasks.find((task) => task.id === taskId);
  if (task) {
    task.completed = true;
  }
};

const undoTask = (taskId) => {
  const task = state.tasks.find((task) => task.id === taskId);
  if (task) {
    task.completed = false;
  }
};

const deleteTask = (taskId) => {
  state.tasks = state.tasks.filter((task) => task.id !== taskId);
};

const filteredTasks = computed(() => {
  if (state.showCompleted) {
    return state.tasks.filter((task) => task.completed);
  } else {
    return state.tasks;
  }
});
</script>
