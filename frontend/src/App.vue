<template>
  <v-app>
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="8" lg="6">
          <v-card class="pa-5 text-center">
            <v-card-title class="text-h4 font-weight-bold">
              Todo App 🗒️
            </v-card-title>

            <v-list class="mt-5">
              <v-list-item-group>
                <v-list-item
                  v-for="todo in todos"
                  :key="todo._id"
                  class="todo-item"
                >
                  <v-list-item-action>
                    <v-btn icon :color="todo.complete ? 'success' : 'grey'" @click="toggleComplete(todo)">
                      <v-icon>{{ todo.complete ? 'mdi-check-circle' : 'mdi-circle-outline' }}</v-icon>
                    </v-btn>
                  </v-list-item-action>

                  <v-list-item-content class="text-container">
                    <v-list-item-title :class="{ 'completed-text': todo.complete }">
                      {{ todo.text }}
                    </v-list-item-title>
                  </v-list-item-content>

                  <v-list-item-action>
                    <v-btn icon color="error" @click="removeTodo(todo._id)">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>
                  </v-list-item-action>
                </v-list-item>
              </v-list-item-group>
            </v-list>

            <v-card-text>
              <v-textarea
                v-model="newTodo"
                label="Enter a new Todo"
                outlined
                clearable
                auto-grow
                @keydown.enter.prevent="addTodo"
              ></v-textarea>

              <v-btn color="primary" @click="addTodo" class="mt-3">
                Add Todo
              </v-btn>
              <v-btn color="secondary" @click="clearAllTodos" class="mt-3 ml-3">
                Clear All
              </v-btn>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      newTodo: "",
      todos: [],
    };
  },
  mounted() {
    this.fetchTodos();
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await axios.get('http://localhost:5000/todos');
        this.todos = response.data;
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    },
    async addTodo() {
      if (this.newTodo.trim().length === 0) {
        return;
      }
      try {
        const response = await axios.post('http://localhost:5000/todos', {
          text: this.newTodo
        });
        this.todos.push(response.data);
        this.newTodo = "";
      } catch (error) {
        console.error('Error adding todo:', error);
      }
    },
    async removeTodo(id) {
      try {
        await axios.delete(`http://localhost:5000/todos/${id}`);
        this.todos = this.todos.filter((todo) => todo._id !== id);
      } catch (error) {
        console.error('Error removing todo:', error);
      }
    },
    async toggleComplete(todo) {
      try {
        const response = await axios.put(`http://localhost:5000/todos/${todo._id}`, {
          complete: !todo.complete
        });
        const index = this.todos.findIndex(t => t._id === todo._id);
        if (index !== -1) {
          this.todos[index] = response.data;
        }
      } catch (error) {
        console.error('Error toggling todo:', error);
      }
    },
    async clearAllTodos() {
      try {
        await axios.delete('http://localhost:5000/todos');
        this.todos = [];
      } catch (error) {
        console.error('Error clearing todos:', error);
      }
    },
  },
};
</script>

<style scoped>
.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-radius: 8px;
  padding: 10px;
  margin-bottom: 8px;
  background-color: rgb(52, 51, 51);
}

.text-container {
  flex-grow: 1;
  display: flex;
  align-items: center;
}

.completed-text {
  text-decoration: line-through;
  color: grey;
}
</style>