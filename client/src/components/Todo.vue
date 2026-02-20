<template>
  <section class="main">
    <p class="lead mb-4">
      Tasks: {{ todoList.length }}
      <span class="text-secondary">(Active: {{ activeCount }}, Completed: {{ completedCount }})</span>
    </p>
    <div class="input-group mb-3 input-group-lg">
      <input
        class="form-control new-todo"
        style="width: auto;"
        autofocus
        autocomplete="off"
        placeholder="What needs to be done?"
        type="text"
        v-model="newTask"
        @keyup.enter="addTask(newTask)"
      />
      <div class="input-group-append">
        <button
          class="btn btn-primary"
          type="button"
          style="padding: 10px 8px;"
          :disabled="!newTask.trim()"
          @click="addTask(newTask)"
        >Add Task</button>
      </div>
    </div>

    <div class="todo-list" v-if="todoList.length > 0" v-cloak>
      <ul class="list-group">
        <li class="list-group-item" :class="{ editing: todo.editing }" v-for="todo in filteredList" :key="todo._id">
          <div v-if="todo.editing">
            <input
              type="text"
              class="form-control"
              autofocus
              autocomplete="off"
              :value="todo.todo"
              @keyup.esc="$event.target.blur()"
              @keyup.enter="editTask(todo._id, $event.target.value, todo.completed, false)"
              @blur="cancelEdit(todo._id, todo.completed)"
              v-focus
            />
          </div>
          <div class="list-group-row"  v-else>
            <div class="d-flex align-items-center">
              <input
                type="checkbox"
                class="checkbox"
                @click="editTask(todo._id, todo.todo, !todo.completed, todo.editing)"
                :checked="todo.completed"
              />
              <label
                class="todo-item"
                @click="editTask(todo._id, todo.todo, todo.completed, true)"
                :class="{ 'completed': todo.completed }"
              >
                <p class="lead">{{ todo.todo }}</p>
                <span style="font-size: .875rem; color: #666;">Creation Date: {{ formatDate(todo.created) }}</span>
              </label>
            </div>


            <button
              type="button"
              class="close"
              aria-label="close"
              @click="deleteTask(todo._id, $event.target)"
            >
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
        </li>
      </ul>

      <button
        @click="visibility='all'"
        :class="['btn btn-outline-primary filter', { active: visibility === 'all' }]"
      >Show All</button>
      <button
        @click="visibility='active'"
        :class="['btn btn-outline-primary filter', { active: visibility === 'active' }]"
      >Show Active</button>
      <button
        @click="visibility='completed'"
        :class="['btn btn-outline-primary filter', { active: visibility === 'completed' }]"
      >Show Completed</button>
      <p class="lead footer-text">click a todo item to edit | created by marcin pawlik</p>
    </div>

    <p v-else class="lead">Looks like you don't have any todos yet! :(</p>
  </section>
</template>

<script>
import ApiClient from "../ApiClient";

const filters = {
  all: function(todos) {
    return todos;
  },
  active: function(todos) {
    return todos.filter(function(todo) {
      return !todo.completed;
    });
  },
  completed: function(todos) {
    return todos.filter(function(todo) {
      return todo.completed;
    });
  }
};

export default {
  name: "Todo",
  data() {
    return {
      todoList: [],
      newTask: "",
      visibility: "all",
      beforeEditCache: ""
    };
  },
  computed: {
    filteredList: function() {
      return filters[this.visibility](this.todoList);
    },
    activeCount: function() {
      return this.todoList.filter(t => !t.completed).length;
    },
    completedCount: function() {
      return this.todoList.filter(t => t.completed).length;
    }
  },
  methods: {
    // Retrieve tasks from API and save in the todoList array
    retrieveTasks: async function() {
      await ApiClient.retrieveTasks()
        .then(todos => {
          this.todoList = todos;
        })
        .catch(err => this.$toastr.e(`An error occurred : ${err}`));
    },
    // Add task to todolist
    addTask: async function(task) {
      if (!/\S/.test(task)) return;
      let created = new Date();
      await ApiClient.addTask(task, created)
        .then(() => {
          this.newTask = ""; // Clear input field
          this.retrieveTasks(); // Reload the array
        })
        .catch(err => this.$toastr.e(`An error occurred : ${err}`));
    },
    // Delete task from todolist
    deleteTask: async function(id, el=null) {
      if(el) el.parentElement.disabled = true; // Disable button after click to prevent 404 error
      await ApiClient.deleteTask(id)
        .then(() => {
          this.retrieveTasks(); // Reload the array
        })
        .catch(err => this.$toastr.e(`An error occurred : ${err}`));
    },
    // Edit task in todolist
    editTask: async function(id, task, completed, editing) {
      this.beforeEditCache = task; // Save the task to edit cache
      if (!/\S/.test(task)) return; // Return if the new task is empty
      await ApiClient.editTask(id, task, completed, editing)
        .then(() => {
          this.retrieveTasks(); // Reload the array
        })
        .catch(err => this.$toastr.e(`An error occurred : ${err}`));
    },
    // Revert the task back to the state in the cache
    cancelEdit: async function(id, completed) {
      if (!/\S/.test(this.beforeEditCache)) return; // Return if the cache is empty
      await ApiClient.editTask(id, this.beforeEditCache, completed, false)
        .then(() => {
          this.retrieveTasks(); // Reload the array
        })
        .catch(err => this.$toastr.e(`An error occurred: ${err}`));
    },
    formatDate: function(dateString) {
      let date = new Date(dateString);
      return new Date(date.getTime() - date.getTimezoneOffset() * 60000)
        .toISOString()
        .split("T")[0];
    }
  },
  // Retrieve tasks when the page is loaded
  async beforeMount() {
    await this.retrieveTasks();
  },
  // Custom directive to focus on input field when the DOM is updated
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      }
    }
  }
};
</script>
