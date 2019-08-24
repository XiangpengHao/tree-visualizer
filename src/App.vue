<template>
  <div id="app">
    <el-select size="small" v-model="subtree" placeholder="subtree index">
      <el-option v-for="i in 255" :key="i" :label="i" :value="i"></el-option>
    </el-select>
    <input id="file-upload" @change="onFileChange" accept="*.json" type="file" />
    <Tree :rawTree="subTreeJson" />
  </div>
</template>

<script>
import Tree from "./components/Tree.vue";

export default {
  name: "app",
  components: {
    Tree
  },
  computed: {
    subTreeJson() {
      return this.userJson !== null ? this.userJson[this.subtree].t : {};
    }
  },
  data() {
    return {
      subtree: 49,
      userJson: null
    };
  },
  methods: {
    onFileChange(e) {
      let self = this;
      let files = e.target.files || e.dataTransfer.files;
      if (files.length === 1) {
        self.file = files[0];
      } else {
        return;
      }
      let reader = new FileReader();
      reader.onload = function(e) {
        let contents = e.target.result;
        self.userJson = JSON.parse(contents);
      };
      reader.readAsText(self.file);
    }
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
