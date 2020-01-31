<template>
  <div id="app">
    <my-canvas />
  </div>
</template>

<script>
import MyCanvas from './components/MyCanvas.vue';

export default {
  name: 'app',
  components: {
    MyCanvas,
  },
};
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
