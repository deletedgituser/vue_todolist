<template>
  <v-container class="pa-5">
    <v-row>
      <v-col cols="12">
        <h1 class="title">Track-Your-Task</h1>
        <v-form ref="form">
          <v-text-field
            v-model="newTodo"
            label="Enter todo"
            required
            clearable
            class="input-field"
          ></v-text-field>

          <v-text-field
            v-model="dueDate"
            label="Due Date"
            type="datetime-local"
            class="input-field"
          ></v-text-field>

          <v-btn
            color="primary"
            @click.prevent="addTodo"
            :disabled="!isFormValid"
            class="add-btn"
          >
            ADD
          </v-btn>
        </v-form>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12" class="d-flex justify-center">
        <v-btn @click="viewDoneList" color="secondary" class="view-btn">
          View Completed
        </v-btn>
        <v-btn @click="viewOngoingList" color="secondary" class="view-btn">
          View Ongoing
        </v-btn>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-list dense>
          <v-list-item-group v-if="currentList === 'ongoing'">
            <v-list-item
              v-for="(todo, index) in ongoingTodos"
              :key="index"
              class="list-item"
            >
              <v-list-item-content v-if="!todo.isEditing">
                <v-checkbox v-model="todo.check" class="checkbox" @change="updateTodoStatus(todo)" />
                <v-list-item-title class="todo-text">{{ todo.text }}</v-list-item-title>
                <v-list-item-subtitle v-if="todo.due">Due: {{ todo.due }}</v-list-item-subtitle>
                <v-btn @click="editTodo(index)" color="primary" class="action-btn">Edit</v-btn>
                <v-btn @click="deleteTodo(index)" color="error" class="action-btn">Delete</v-btn>
              </v-list-item-content>

              <v-list-item-content v-else>
                <v-text-field v-model="editText" class="edit-field" />
                <v-text-field v-model="editDue" type="datetime-local" class="edit-field" />
                <v-btn @click="saveEdit(index)" color="success" class="action-btn">Save</v-btn>
                <v-btn @click="cancelEdit(index)" color="error" class="action-btn">Cancel</v-btn>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>

          <v-list-item-group v-if="currentList === 'done'">
            <v-list-item
              v-for="(todo, index) in doneTodos"
              :key="index"
              class="list-item"
            >
              <v-list-item-content>
                <v-list-item-title class="todo-text">{{ todo.text }}</v-list-item-title>
                <v-list-item-subtitle>Completed</v-list-item-subtitle>
                <v-list-item-subtitle v-if="todo.due">Due: {{ todo.due }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>

        <v-btn
          v-if="todos.length"
          color="error"
          @click="clearAll"
          class="clear-all-btn"
        >
          Clear All
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      newTodo: "",
      editText: "",
      editDue: "",
      dueDate: "",
      todos: [],
      formValid: false,
      currentList: 'ongoing', // 'ongoing' or 'done'
    };
  },
  computed: {
    ongoingTodos() {
      return this.todos.filter(todo => !todo.check);
    },
    doneTodos() {
      return this.todos.filter(todo => todo.check);
    },
    isFormValid() {
        // Check if both newTodo and dueDate are not empty
        return this.newTodo.trim() !== "" && this.dueDate.trim() !== "";
    },
  },
  methods: {
    addTodo() {
      if (this.$refs.form.validate()) {
        const formattedDueDate = this.dueDate
          ? new Date(this.dueDate).toLocaleString('en-US', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
              hour: 'numeric',
              minute: 'numeric',
              hour12: true,
            })
          : null;

        const newTodo = {
          text: this.newTodo,
          isEditing: false,
          check: false,
          due: formattedDueDate
        };

        this.todos.push(newTodo);
        this.saveTodosToLocalStorage(); // Save to local storage

        this.newTodo = "";
        this.dueDate = "";

        // Ensure the view shows the ongoing list after adding a new task
        this.currentList = 'ongoing';
        this.saveCurrentListToLocalStorage(); // Save current list view state
      }
    },
    updateTodoStatus(updatedTodo) {
      // Find the index of the updated todo
      const index = this.todos.findIndex(todo => todo === updatedTodo);
      if (index !== -1) {
        this.todos[index] = { ...updatedTodo }; // Create a new reference to trigger reactivity
        this.saveTodosToLocalStorage(); // Save to local storage
      }
    },
    deleteTodo(index) {
      this.todos.splice(index, 1);
      this.saveTodosToLocalStorage(); // Save to local storage
    },
    editTodo(index) {
      this.todos[index].isEditing = true;
      this.editText = this.todos[index].text;
    },
    saveEdit(index) {
      if (this.editText.trim() !== "" && this.editDue.trim() !== "") {
        const formattedDueDate = this.editDue
          ? new Date(this.editDue).toLocaleString('en-US', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
              hour: 'numeric',
              minute: 'numeric',
              hour12: true,
            })
          : null;
        this.todos[index].text = this.editText;
        this.todos[index].due = formattedDueDate;
        this.todos[index].isEditing = false;
        this.editText = "";
        this.saveTodosToLocalStorage(); // Save to local storage
      }
    },
    cancelEdit(index) {
      this.todos[index].isEditing = false;
      this.editText = "";
    },
    clearAll() {
      this.todos = [];
      this.saveTodosToLocalStorage(); // Save to local storage
    },
    viewDoneList() {
      this.currentList = 'done';
      this.saveCurrentListToLocalStorage(); // Save current list view state
    },
    viewOngoingList() {
      this.currentList = 'ongoing';
      this.saveCurrentListToLocalStorage(); // Save current list view state
    },
    saveTodosToLocalStorage() {
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    loadTodosFromLocalStorage() {
      const storedTodos = localStorage.getItem('todos');
      if (storedTodos) {
        this.todos = JSON.parse(storedTodos);
      }
    },
    saveCurrentListToLocalStorage() {
      localStorage.setItem('currentList', this.currentList);
    },
    loadCurrentListFromLocalStorage() {
      const storedList = localStorage.getItem('currentList');
      if (storedList) {
        this.currentList = storedList;
      }
    }
  },
  created() {
    this.loadTodosFromLocalStorage(); // Load todos when component is created
    this.loadCurrentListFromLocalStorage(); // Load current list view state
  },
};
</script>

<style scoped>
.title {
  color: #333;
  text-align: center;
  margin-bottom: 20px;
}

.input-field,
.edit-field {
  background-color: #f9f9f9;
  color: #333;
  border: 1px solid #ccc;
}

.add-btn,
.action-btn,
.clear-all-btn,
.view-btn {
  margin: 10px;
}

.add-btn {
  background-color: #007bff;
  color: white;
}

.checkbox {
  color: #333;
}

.list-item {
  background-color: #e9ecef; /* Light grey background for the list items */
  border-radius: 8px;
  margin-bottom: 10px;
  color: #333;
  word-break: break-word; /* Break long words and wrap text */
  overflow-wrap: break-word; /* Handle overflow and wrapping */
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 10px; /* Add padding for better readability */
}

.todo-text {
  color: #333;
  white-space: normal; /* Allow text to wrap */
}

.clear-all-btn {
  background-color: #dc3545;
  color: white;
}

.view-btn {
  background-color: #6c757d;
  color: white;
}
</style>
