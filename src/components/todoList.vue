<script setup>
import { Cross, MenuIcon, Moon, Sun, X } from "lucide-vue-next";
import { ref, onMounted, watch } from "vue";

/* ---------------- STATE ---------------- */
const task = ref("");
const todos = ref([]);
const editingId = ref(null);
const isDark = ref(false);
const filter = ref("all");
const showDeleteModal = ref(false);
const deleteId = ref(null);
const draggingTodo = ref(null);
const showMenu = ref(false);
const showTrash = ref(false);
const trash = ref([]);

/* ---------------- LOAD ---------------- */
onMounted(() => {
  const savedTodos = localStorage.getItem("todos");
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos);
    sortTodos();
  }

  const savedTrash = localStorage.getItem("trash");
  if (savedTrash) {
    trash.value = JSON.parse(savedTrash);
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
watch(
  trash,
  () => {
    localStorage.setItem("trash", JSON.stringify(trash.value));
  },
  { deep: true },
);
function toggleMenu() {
  showMenu.value = !showMenu.value;
}

function onDragStart(todo) {
  draggingTodo.value = todo;
}

function onDrop(todo) {
  if (!draggingTodo.value) return;

  const fromIndex = todos.value.findIndex(
    (t) => t.id === draggingTodo.value.id,
  );
  const toIndex = todos.value.findIndex((t) => t.id === todo.id);

  const movedItem = todos.value.splice(fromIndex, 1)[0];
  todos.value.splice(toIndex, 0, movedItem);

  draggingTodo.value = null;
}

/* ---------------- THEME ---------------- */
function toggleTheme() {
  isDark.value = !isDark.value;
}

/* ---------------- SORT ---------------- */
function sortTodos() {
  todos.value.sort((a, b) => a.completed - b.completed);
}

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
  const item = todos.value.find((t) => t.id === deleteId.value);
  if (!item) return;

  trash.value.push(item);
  todos.value = todos.value.filter((t) => t.id !== deleteId.value);

  showDeleteModal.value = false;
  deleteId.value = null;
}

function cancelDelete() {
  showDeleteModal.value = false;
  deleteId.value = null;
}

function restoreTodo(id) {
  const item = trash.value.find((t) => t.id === id);
  if (!item) return;

  todos.value.push(item);
  trash.value = trash.value.filter((t) => t.id !== id);
}

function deleteForever(id) {
  trash.value = trash.value.filter((t) => t.id !== id);
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
    class="flex items-center justify-center min-h-screen px-4 transition"
  >
    <!-- CARD -->
    <div
      :class="isDark ? 'bg-gray-800' : 'bg-white'"
      class="relative w-full max-w-md p-6 shadow-2xl rounded-2xl"
    >
      <!-- HEADER -->
      <div class="relative flex items-center justify-between mb-5">
        <h1 class="text-2xl font-bold">📝 To-Do List</h1>

        <button @click="toggleTheme" class="p-2 rounded-full">
          <Moon v-if="isDark" class="w-5 h-5" />
          <Sun v-else class="w-5 h-5" />
        </button>

        <!-- MENU -->
        <div class="relative">
          <button @click="toggleMenu" class="p-2">
            <MenuIcon v-if="!showMenu" class="w-6 h-6" />
            <X v-else class="w-6 h-6" />
          </button>

          <div
            v-if="showMenu"
            class="absolute right-0 z-50 w-56 mt-2 overflow-hidden bg-white shadow-xl dark:bg-gray-800 rounded-xl"
          >
            <!-- HEADER -->
            <div
              class="flex items-center justify-between px-4 py-3 border-b border-gray-200 dark:border-gray-700"
            >
              <span class="font-semibold text-gray-800 dark:text-white"
                >Menu</span
              >
              <button @click="showMenu = false">
                <X class="w-4 h-4" />
              </button>
            </div>

            <!-- ITEMS -->
            <button
              @click="
                showTrash = true;
                showMenu = false;
              "
              class="w-full px-4 py-3 text-left text-red-500 hover:bg-gray-100 dark:hover:bg-gray-700"
            >
              🗑 Recently Deleted ({{ trash.length }})
            </button>
          </div>
        </div>
      </div>

      <!-- FILTER -->
      
      <div class="flex gap-2 mb-4">
        <button
          @click="filter = 'all'"
          :class="
            filter === 'all'
              ? 'bg-blue-500 text-white'
              : isDark
                ? 'bg-gray-700 text-white'
                : 'bg-gray-300 text-black'
          "
          class="px-3 py-1 rounded"
        >
          All
        </button>

        <button
          @click="filter = 'pending'"
          :class="
            filter === 'pending'
              ? 'bg-yellow-500 text-white'
              : isDark
                ? 'bg-gray-700 text-white'
                : 'bg-gray-300 text-black'
          "
          class="px-3 py-1 rounded"
        >
          Pending
        </button>

        <button
          @click="filter = 'completed'"
          :class="
            filter === 'completed'
              ? 'bg-green-500 text-white'
              : isDark
                ? 'bg-gray-700 text-white'
                : 'bg-gray-300 text-black'
          "
          class="px-3 py-1 rounded"
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
          class="flex-1 px-3 py-2 border rounded-lg"
        />

        <button class="px-4 text-white bg-red-500 rounded-lg">
          {{ editingId === null ? "ADD" : "SAVE" }}
        </button>
      </form>

      <!-- LIST -->
      <ul class="space-y-3">
        <li
          v-for="todo in getTodo()"
          :key="todo.id"
          draggable="true"
          @dragstart="onDragStart(todo)"
          @dragover.prevent
          @drop="onDrop(todo)"
          class="flex items-center justify-between p-3 transition rounded-lg shadow-sm"
          :class="
            isDark
              ? 'bg-gray-700 text-white hover:bg-gray-600'
              : 'bg-gray-100 text-gray-900 hover:bg-gray-200'
          "
        >
          <div class="flex items-center gap-2">
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleTodo(todo)"
            />

            <span :class="todo.completed ? 'line-through text-gray-400' : ''">
              {{ todo.text }}
            </span>
          </div>

          <div class="flex gap-2">
            <button @click="editTodo(todo)" class="text-green-500">Edit</button>
            <button @click="askDelete(todo.id)" class="text-red-500">
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
          class="flex flex-col gap-4 p-6 text-center bg-white dark:bg-gray-800 rounded-xl w-80"
        >
          <h2 class="text-lg font-bold">Delete Task?</h2>
          <p class="text-sm text-gray-500">This action cannot be undone.</p>

          <div class="flex justify-center gap-3">
            <button @click="cancelDelete" class="px-4 py-2 bg-gray-400 rounded">
              Cancel
            </button>

            <button
              @click="confirmDelete"
              class="px-4 py-2 text-white bg-red-500 rounded"
            >
              Delete
            </button>
          </div>
        </div>
      </div>

      <!-- TRASH MODAL -->
      <div
        v-if="showTrash"
        class="fixed inset-0 z-50 flex items-center justify-center bg-black/60"
      >
        <div class="p-5 bg-white dark:bg-gray-800 rounded-xl w-96">
          <h2 class="mb-4 text-lg font-bold">Recently Deleted</h2>

          <div v-if="trash.length === 0" class="text-gray-400">
            No deleted tasks
          </div>

          <ul class="space-y-2 overflow-auto max-h-60">
            <li
              v-for="item in trash"
              :key="item.id"
              class="flex items-center justify-between p-3 rounded-lg"
              :class="
                isDark ? 'bg-gray-700 text-white' : 'bg-gray-100 text-gray-900'
              "
            >
              <span>{{ item.text }}</span>

              <div class="flex gap-2">
                <button @click="restoreTodo(item.id)" class="text-green-500">
                  Restore
                </button>

                <button @click="deleteForever(item.id)" class="text-red-500">
                  Delete
                </button>
              </div>
            </li>
          </ul>

          <button
            @click="showTrash = false"
            class="w-full py-2 mt-4 text-white bg-red-500 rounded"
          >
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
