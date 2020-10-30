<template></template>

<script>
/* eslint-disable */
import * as d3 from "../../../lib/d3";
export default {
  name: "MapIntro",
  props: {},
  data() {
    return {};
  },
  mounted() {},
  method: {
    drawMap: function () {
     console.log("setting maps opacity");
          d3.selectAll(".country").attr("opacity", 1);
          var totalLength = d3.selectAll(".country").node().getTotalLength();
          console.log(totalLength)
          d3.selectAll(".country")
           .attr("opacity", 1)
            .attr("stroke-dasharray", totalLength + " " + totalLength)
            .attr("stroke-dashoffset", totalLength)
            .transition()
            .duration(4000)
            .ease(d3.easeLinear)
            .attr("stroke-dashoffset", 0)
    },
  },
};
</script>

<style scoped>
</style>