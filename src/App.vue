<template>
  <div id="app">
    <el-select size="small" v-model="subtree" placeholder="subtree index">
      <el-option v-for="i in 255" :key="i" :label="i" :value="i"></el-option>
    </el-select>
    <input id="file-upload" @change="onFileChange" accept="*.json" type="file" />
    <el-card class="info">{{nodeData}}</el-card>
    <Tree :rawTree="subTreeJson" v-on:nodeDetail="nodeDetail" />
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
    nodeDetail(d) {
      if (!d.data.ad) {
        return;
      }
      console.log(d);
      let subnodeData = this.countSubNode(d.data);

      let nodeString = `node addr: 0x${d.data.ad.toString(16)}, node level: ${
        d.data.nl
      }, tree level: ${d.data.tl}, children count: ${
        d.data.cd.length
      }, subnode count: ${subnodeData[0]}(internal), ${subnodeData[1]}(leaf)`;
      this.nodeData = nodeString;
    },
    siblingDis(d) {
      let parent = d.parent;
    },
    countSubNode(d) {
      let count = [0, 0];
      for (let i = 0; i < d.cd.length; i += 1) {
        if (!d.cd[i].cd) {
          count[1] += 1;
        } else {
          let sub = this.countSubNode(d.cd[i]);
          count[0] += 1;
          count[0] += sub[0];
          count[1] += sub[1];
        }
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
  width: 25%;
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
