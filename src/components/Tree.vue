<template>
  <div id="tree"></div>
</template>

<script>
import * as d3 from "d3";
import colorMapping from "../assets/color_mapping.json";
const margin = { top: 20, right: 90, bottom: 30, left: 90 };
const width = 960 - margin.left - margin.right;
const height = 1000 - margin.top - margin.bottom;

export default {
  name: "Tree",
  props: {
    rawTree: Object
  },
  data() {
    return {
      treemap: null,
      root: null,
      svg: null,
      duration: 500,
      i: 0
    };
  },
  watch: {
    rawTree() {
      this.rerender();
    }
  },
  mounted() {
    // append the svg object to the body of the page
    // appends a 'group' element to 'svg'
    // moves the 'group' element to the top left margin
    this.svg = d3
      .select("#tree")
      .append("svg")
      .attr("width", width + margin.right + margin.left)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    // declares a tree layout and assigns the size
    this.treemap = d3.tree().size([height, width]);
    this.rerender();
  },
  methods: {
    rerender() {
      // Assigns parent, children, height, depth
      this.root = d3.hierarchy(this.rawTree, function(d) {
        return d.cd;
      });
      this.root.x0 = height / 2;
      this.root.y0 = 0;

      // Collapse after the second level
      this.root.children.forEach(this.collapse);

      this.update(this.root);
    },
    update(source) {
      let self = this;
      // Assigns the x and y position for the nodes
      var treeData = this.treemap(this.root);

      // Compute the new tree layout.
      var nodes = treeData.descendants(),
        links = treeData.descendants().slice(1);

      // Normalize for fixed-depth.
      nodes.forEach(function(d) {
        d.y = d.depth * 180;
      });

      // ****************** Nodes section ***************************

      // Update the nodes...
      var node = this.svg.selectAll("g.node").data(nodes, function(d) {
        return d.id || (d.id = ++self.i);
      });

      // Enter any new modes at the parent's previous position.
      var nodeEnter = node
        .enter()
        .append("g")
        .attr("class", "node")
        .attr("transform", function() {
          return "translate(" + source.y0 + "," + source.x0 + ")";
        })
        .on("click", this.click);

      // Add Circle for the nodes
      nodeEnter
        .append("circle")
        .attr("class", "node")
        .attr("r", 1e-6)
        .style("fill", function(d) {
          if (d.data.tl === undefined) {
            let parent = d.parent;
            return colorMapping[parent.data.tl][parent.data.nl + 1];
          }
          return colorMapping[d.data.tl][d.data.nl];
        });

      // Add labels for the nodes
      nodeEnter
        .append("text")
        .attr("dy", ".35em")
        .attr("x", function(d) {
          return d.children || d._children ? -13 : 13;
        })
        .attr("text-anchor", function(d) {
          return d.children || d._children ? "end" : "start";
        })
        .text(function(d) {
          if (d.data.ad) {
            return `0x${d.data.ad.toString(16)}`;
          } else {
            return d.data;
          }
        });

      // UPDATE
      var nodeUpdate = nodeEnter.merge(node);

      // Transition to the proper position for the node
      nodeUpdate
        .transition()
        .duration(this.duration)
        .attr("transform", function(d) {
          return "translate(" + d.y + "," + d.x + ")";
        });

      // Update the node attributes and style
      nodeUpdate
        .select("circle.node")
        .attr("r", 10)
        .style("fill", function(d) {
          if (d.data.tl === undefined) {
            let parent = d.parent;
            return colorMapping[parent.data.tl][parent.data.nl + 1];
          }
          return colorMapping[d.data.tl][d.data.nl];
        })
        .attr("cursor", "pointer");

      // Remove any exiting nodes
      var nodeExit = node
        .exit()
        .transition()
        .duration(this.duration)
        .attr("transform", function() {
          return "translate(" + source.y + "," + source.x + ")";
        })
        .remove();

      // On exit reduce the node circles size to 0
      nodeExit.select("circle").attr("r", 1e-6);

      // On exit reduce the opacity of text labels
      nodeExit.select("text").style("fill-opacity", 1e-6);

      // ****************** links section ***************************

      // Update the links...
      var link = this.svg.selectAll("path.link").data(links, function(d) {
        return d.id;
      });

      // Enter any new links at the parent's previous position.
      var linkEnter = link
        .enter()
        .insert("path", "g")
        .attr("class", "link")
        .attr("d", function() {
          var o = { x: source.x0, y: source.y0 };
          return self.diagonal(o, o);
        });

      // UPDATE
      var linkUpdate = linkEnter.merge(link);

      // Transition back to the parent element position
      linkUpdate
        .transition()
        .duration(this.duration)
        .attr("d", function(d) {
          return self.diagonal(d, d.parent);
        });

      // Remove any exiting links
      link
        .exit()
        .transition()
        .duration(this.duration)
        .attr("d", function() {
          var o = { x: source.x, y: source.y };
          return self.diagonal(o, o);
        })
        .remove();

      // Store the old positions for transition.
      nodes.forEach(function(d) {
        d.x0 = d.x;
        d.y0 = d.y;
      });
    },
    // Collapse the node and all it's children
    collapse(d) {
      if (d.children) {
        d._children = d.children;
        d._children.forEach(this.collapse);
        d.children = null;
      }
    },
    // Creates a curved (diagonal) path from parent to the child nodes
    diagonal(s, d) {
      let path = `M ${s.y} ${s.x}
            C ${(s.y + d.y) / 2} ${s.x},
              ${(s.y + d.y) / 2} ${d.x},
              ${d.y} ${d.x}`;
      return path;
    },
    // Toggle children on click.
    click(d) {
      if (d.children) {
        d._children = d.children;
        d.children = null;
      } else {
        d.children = d._children;
        d._children = null;
      }
      this.update(d);
    },
    hasChild(d) {
      return d._children || d.children;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style>
.node circle {
  fill: #fff;
  stroke: rgba(1, 1, 1, 0.2);
  stroke-width: 3px;
}

.node text {
  font: 12px consolas;
}

.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 2px;
}
</style>
<style scoped>
</style>
