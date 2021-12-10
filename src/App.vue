<template>
    <SampleFormComponent />
</template>

<script>
import SampleFormComponent from './components/SampleFormComponent.vue'

export default {
  name: 'App',
  components: {
    SampleFormComponent
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
