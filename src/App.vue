<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" height="100">
    <CertBuilder />
  </div>
</template>

<script>
import CertBuilder from './CertBuilder.vue'

export default {
  name: 'App',
  components: {
    CertBuilder
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
  margin-top: 30px;
}
</style>
