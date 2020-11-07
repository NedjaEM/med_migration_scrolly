<template>
<div id="map"></div>
</template>

<script>
/* eslint-disable */
import * as d3 from "../../../lib/d3";
import map_json from "../../public/Data/total.geo.json";

export default {
  name: "App",
  components: {
    // MapIntro,
  },
  data: function () {
    return {
      width: window.innerWidth * 1.2,
      height: window.innerHeight * 1.4,
      legend_keys: ["Departing", "Arriving"],
      svg: d3
        .select("#map")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height),
    };
  },
  mounted() {
 
    // this.drawInitial();
  },
  method: {
     drawMap: function () {
      console.log("setting maps opacity");
      d3.selectAll(".country").attr("opacity", 1);
      var totalLength = d3.selectAll(".country").node().getTotalLength();
      console.log(totalLength);
      d3.selectAll(".country")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength + " " + totalLength)
        .attr("stroke-dashoffset", totalLength)
        .transition()
        .duration(2000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

    },
  },
};
</script>

<style scoped>
</style>