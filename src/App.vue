<template>
  <TimeDisplay/>
</template>

<script>
import TimeDisplay from './components/TimeDisplay.vue'

export default {
  name: 'App',
  components: {
    TimeDisplay
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
