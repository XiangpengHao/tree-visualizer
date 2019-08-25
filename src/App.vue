<template>
  <div id="app">
    <el-select size="small" v-model="subtree" placeholder="subtree index">
      <el-option v-for="i in 255" :key="i" :label="i" :value="i"></el-option>
    </el-select>
    <input id="file-upload" @change="onFileChange" accept="*.json" type="file" />
    <el-card class="info">{{nodeData}}</el-card>
    <Tree :rawTree="subTreeJson" v-on:nodeDetail="computeNode" />
  </div>
</template>

<script>
import Tree from "./components/Tree.vue";
import large_json from "./assets/large_insert2.json";

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
      userJson: large_json,
      nodeData: "alt + click on node to see node detail"
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
    },
    computeNode(d) {
      if (!d.data.ad) {
        return;
      }
      let nodeString = `node addr: 0x${d.data.ad.toString(16)}, node level: ${
        d.data.nl
      }, tree level: ${d.data.tl}, children count: ${
        d.data.cd.length
      }, subnode count: ${this.countSubNode(d.data)}`;
      this.nodeData = nodeString;
    },
    countSubNode(d) {
      if (!d.cd) {
        return 0;
      }
      let count = d.cd.length;
      for (let i = 0; i < d.cd.length; i += 1) {
        count += this.countSubNode(d.cd[i]);
      }
      return count;
    }
  }
};
</script>
<style scoped>
.info {
  position: fixed;
  right: 1em;
  top: 4em;
  width: 20%;
  height: 20%;
}
</style>

<style>
#app {
  font-family: consolas;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
