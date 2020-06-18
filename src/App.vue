<template>
  <div id="app">
    <Chart/>
  </div>
</template>

<script>
  import Chart from './components/Chart.vue'

  export default {
    name: 'App',
    components: {
      Chart
    },
    data: function() {
      return {
      };
    },

    mounted() {
      console.log("App loaded");
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
  }
  html, body {
    overflow: hidden;
  }
  body {
    margin: 8px;
    padding: 0px;
  }

</style>
