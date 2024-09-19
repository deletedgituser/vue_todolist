<template>
  <v-app>
    <v-main>
      <MainTodo/>
    </v-main>
  </v-app>
</template>

<script>
import MainTodo from './components/MainTodo.vue'

export default {
  name: 'App',

  components: {
    MainTodo
  }
}
</script>
