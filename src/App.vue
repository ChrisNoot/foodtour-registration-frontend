<template>
  <h1 class="main-title">Ben's Food Tour Amsterdam</h1>
  <div id="app">
    <CalenderMainpage/>
  </div>
</template>

<script>
import CalenderMainpage from './components/CalenderMainpage.vue'

export default {
  name: 'App',
  components: {
    CalenderMainpage
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
