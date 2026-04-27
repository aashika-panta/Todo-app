<script setup>
import { Moon, Sun } from "lucide-vue-next";
import { ref, onMounted, watch } from "vue";

/* ---------------- STATE ---------------- */
const task = ref("");
const todos = ref([]);
const editingId = ref(null);
const isDark = ref(false);
const filter = ref("all");
const showDeleteModal = ref(false);
const deleteId = ref(null);

/* ---------------- LOAD ---------------- */
onMounted(() => {
  const saved = localStorage.getItem("todos");
  if (saved) {
    todos.value = JSON.parse(saved);
    sortTodos();
  }
});

/* ---------------- AUTO SAVE ---------------- */
watch(
  todos,
  () => {
    localStorage.setItem("todos", JSON.stringify(todos.value));
  },
  { deep: true },
);

/* ---------------- THEME ---------------- */
function toggleTheme() {
  isDark.value = !isDark.value;
}

/* ---------------- SORT  ---------------- */
function sortTodos() {
  todos.value.sort((a, b) => a.completed - b.completed);
}

// // add only
// function submitTodo() {
//   if (!task.value.trim()) return;

//   todos.value.push({
//     id: Date.now(),
//     text: task.value,
//     completed: false,
//   });

//   task.value = "";
// }

/* ---------------- ADD / EDIT ---------------- */

function submitTodo() {
  if (!task.value.trim()) return;

  if (editingId.value === null) {
    todos.value.push({
      id: Date.now(),
      text: task.value,
      completed: false,
    });
  } else {
    const todo = todos.value.find((t) => t.id === editingId.value);
    if (todo) todo.text = task.value;
    editingId.value = null;
  }

  task.value = "";
  sortTodos();
}

function toggleTodo(todo) {
  todo.completed = !todo.completed;

  sortTodos();
}

/* ---------------- EDIT ---------------- */
function editTodo(todo) {
  task.value = todo.text;
  editingId.value = todo.id;
}

/* ---------------- DELETE ---------------- */
function askDelete(id) {
  deleteId.value = id;
  showDeleteModal.value = true;
}

function confirmDelete() {
  todos.value = todos.value.filter((t) => t.id !== deleteId.value);
  showDeleteModal.value = false;
  deleteId.value = null;
}

function cancelDelete() {
  showDeleteModal.value = false;
  deleteId.value = null;
}
/* ---------------- FILTER ---------------- */
function getTodo() {
  if (filter.value === "pending") {
    return todos.value.filter((t) => !t.completed);
  }

  if (filter.value === "completed") {
    return todos.value.filter((t) => t.completed);
  }

  return todos.value;
}
</script>

<template>
  <div
    :class="isDark ? 'bg-gray-900 text-white' : 'bg-gray-100 text-black'"
    class="flex items-center justify-center min-h-screen px-4 transition duration-300"
  >
    <!-- CARD -->
    <div
      :class="isDark ? 'bg-gray-800' : 'bg-white'"
      class="w-full max-w-md p-6 transition shadow-2xl rounded-2xl"
    >
      <!-- HEADER -->
      <div class="flex items-center justify-between mb-5">
        <h1 class="text-2xl font-bold">📝 To-Do List</h1>

        <button
          @click="toggleTheme"
          class="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700"
        >
          <Moon v-if="isDark" class="w-5 h-5" />
          <Sun v-else class="w-5 h-5" />
        </button>
      </div>

      <!-- FILTER -->
      <div class="flex gap-2 mb-4">
        <button
          @click="filter = 'all'"
          :class="
            filter === 'all'
              ? 'bg-blue-500 text-white'
              : isDark
                ? 'bg-gray-700'
                : 'bg-gray-200'
          "
          class="px-3 py-1 transition rounded"
        >
          All
        </button>

        <button
          @click="filter = 'pending'"
          :class="
            filter === 'pending'
              ? 'bg-yellow-500 text-white'
              : isDark
                ? 'bg-gray-700'
                : 'bg-gray-200'
          "
          class="px-3 py-1 transition rounded"
        >
          Pending
        </button>

        <button
          @click="filter = 'completed'"
          :class="
            filter === 'completed'
              ? 'bg-green-500 text-white'
              : isDark
                ? 'bg-gray-700'
                : 'bg-gray-200'
          "
          class="px-3 py-1 transition rounded"
        >
          Completed
        </button>
      </div>

      <!-- INPUT -->
      <form @submit.prevent="submitTodo" class="flex gap-2 mb-4">
        <input
          v-model="task"
          type="text"
          placeholder="Enter task..."
          :class="
            isDark
              ? 'bg-gray-700 text-white border-gray-600'
              : 'bg-gray-100 text-black border-gray-300'
          "
          class="flex-1 px-3 py-2 border rounded-lg outline-none"
        />

        <button
          class="px-4 text-white transition bg-red-500 rounded-lg hover:bg-red-600"
        >
          {{ editingId === null ? "ADD" : "SAVE" }}
        </button>
      </form>

      <!-- LIST -->
      <ul class="space-y-3">
        <li
          v-for="todo in getTodo()"
          :key="todo.id"
          :class="isDark ? 'bg-gray-700 text-white' : 'bg-gray-100 text-black'"
          class="flex items-center justify-between p-3 transition rounded-lg shadow-sm"
        >
          <!-- LEFT -->
          <div class="flex items-center gap-2">
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="
                () => {
                  todo.completed = !todo.completed;
                  sortTodos();
                }
              "
              class="w-4 h-4"
            />

            <span :class="todo.completed ? 'line-through text-gray-400' : ''">
              {{ todo.text }}
            </span>
          </div>

          <!-- RIGHT -->
          <div class="flex gap-2">
            <button
              @click="editTodo(todo)"
              class="text-green-500 hover:text-green-700"
            >
              Edit
            </button>

            <button
              @click="askDelete(todo.id)"
              class="text-red-500 hover:text-red-700"
            >
              Delete
            </button>
          </div>
        </li>
      </ul>
      <!-- DELETE MODAL -->
      <div
        v-if="showDeleteModal"
        class="fixed inset-0 z-50 flex items-center justify-center bg-black/50"
      >
        <div
          class="flex flex-col gap-4 p-6 text-center bg-white shadow-xl dark:bg-gray-800 rounded-xl w-80"
        >
          <h2 class="mb-3 text-lg font-bold">Delete Task?</h2>

          <p class="mb-5 text-sm text-gray-500">
            This action cannot be undone.
          </p>

          <div class="flex justify-center gap-3">
            <button
              @click="cancelDelete"
              class="px-4 py-2 bg-gray-400 rounded hover:bg-gray-600"
            >
              Cancel
            </button>

            <button
              @click="confirmDelete"
              class="px-4 py-2 text-white bg-red-500 rounded hover:bg-red-600"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
