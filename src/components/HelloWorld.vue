<template>
  <div id="cy"></div>
</template>

<script>
import cytoscape from "cytoscape";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {};
  },
  mounted() {
    var cy = (window.cy = cytoscape({
      container: document.getElementById("cy"),

      boxSelectionEnabled: false,

      style: [
        {
          selector: "node",
          css: {
            content: "data(id)",
            "text-valign": "center",
            "text-halign": "center"
          }
        },
        {
          selector: ":parent",
          css: {
            "text-valign": "top",
            "text-halign": "center"
          }
        },
        {
          selector: "edge",
          css: {
            "curve-style": "bezier",
            "target-arrow-shape": "triangle"
          }
        }
      ],

      elements: {
        nodes: [
          { data: { id: "a", parent: "b" }, position: { x: 215, y: 85 } },
          { data: { id: "b" } },
          { data: { id: "c", parent: "b" }, position: { x: 300, y: 85 } },
          { data: { id: "d" }, position: { x: 215, y: 175 } },
          { data: { id: "e" } },
          { data: { id: "f", parent: "e" }, position: { x: 300, y: 175 } }
        ],
        edges: [
          { data: { id: "ad", source: "a", target: "d" } },
          { data: { id: "eb", source: "e", target: "b" } }
        ]
      },

      layout: {
        name: "preset",
        padding: 5
      }
    }));
  },
  methods: {}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#cy {
  height: 100%;
  width: 100%;
  position: absolute;
  left: 0;
  top: 0;
}
</style>
