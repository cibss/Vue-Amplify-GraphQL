<template>
  <amplify-authenticator>
    <div id="app">
      <h1>Todo App</h1>
      <input type="text" v-model="name" placeholder="Todo name">
      <input type="text" v-model="description" placeholder="Todo description">
      <button v-on:click="createTodo">Create Todo</button>
      <div v-for="item in todos" :key="item.id">
        <h3>{{ item.name }}</h3>
        <p>{{ item.description }}</p>
      </div>
    </div>
    <amplify-sign-out></amplify-sign-out>
  </amplify-authenticator>
</template>

<script>
import { API } from 'aws-amplify'
import { createTodo } from './graphql/mutations'
import { listTodos } from './graphql/queries'
import { onCreateTodo } from './graphql/subscriptions'

export default {
  name: 'App',
  // async created() {
  //   this.getTodos()
  // },
  data() {
    return {
      name: '',
      description: ''
    }
  },
  created() {
    this.getTodos()
    this.subscribe()
  },
  methods: {
    async createTodo() {
      const { name, description }= this
      if (!name || !description) return
      const todo = { name, description }
      await API.graphql({
        query: createTodo,
        variables: {input: todo}
      })
      this.name = ''
      this.description = ''
    },
    async getTodos() {
      const todos = await API.graphql({
        query: listTodos
      })
      this.todos = todos.data.listTodos.items
    },
    subscribe() {
      API.graphql({ query: onCreateTodo })
        .subscribe({
          next: (eventData) => {
            let todo = eventData.value.data.onCreateTodo
            if (this.todos.some(item => item.name === todo.name)) return
            this.todos = [...this.todos, todo]
          }
        })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
