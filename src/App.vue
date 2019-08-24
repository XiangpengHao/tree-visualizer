<template>
  <div id="app">
    <Tree msg="" />
  </div>
</template>

<script>
import Tree from "./components/Tree.vue";

export default {
  name: "app",
  components: {
    Tree
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
