<template>
  <div id="app">
    <el-select size="small" v-model="subtree" placeholder="subtree index">
      <el-option v-for="i in 255" :key="i" :label="i" :value="i"></el-option>
    </el-select>
    <Tree :rawTree="subTreeJson" />
  </div>
</template>

<script>
import Tree from "./components/Tree.vue";
import large_insert from "./assets/large_insert2.json";

export default {
  name: "app",
  components: {
    Tree
  },
  computed: {
    subTreeJson() {
      return large_insert[this.subtree].t;
    }
  },
  data() {
    return {
      subtree: 49
    };
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
