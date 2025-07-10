<template>
  <h1 class="main-title">Ben's Food Tour Amsterdam</h1>
  <p class="subtitle">Eat your way through Amsterdam's best-kept culinary secrets with 8 tastings!</p>  <div id="app">
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

.subtitle {
  background: darkgray;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 500;
}
</style>
