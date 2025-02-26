<template>
  <div id="app">
    <connectMe />
  </div>
</template>

<script>
import connectMe from './components/connectMe.vue';

export default {
  components: {
    connectMe,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>