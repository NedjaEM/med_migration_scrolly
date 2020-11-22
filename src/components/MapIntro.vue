<template>
  <div id="graphic">
    <div id="sections">
      <Story :state="state" :active_index="active_index"></Story>
    </div>
    <div id="map"></div>
  </div>
</template>

<script>
/* eslint-disable */
import * as d3 from "../../../lib/d3";
import map_json from "../../public/Data/map.geo.json";
import Story from "./Story.vue";

export default {
  name: "App",
  components: {
    Story
  },
  data: function () {
    return {
      width: window.innerWidth * 1.2,
      height: window.innerHeight * 1.4,
      state: "",
      loadData: [],
      active_index: 0,
      // scroll: this.scroller().container(d3.select("#graphic")),
      legend_keys: ["Departing", "Arriving"],
      svg: d3
        .select("#map")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height),
    };
  },
  mounted() {
    this.drawInitial();

    scroll = this.scroller().container(d3.select("#graphic"));
    scroll();
    let lastIndex,
      activeIndex = 0;

    var scroll_functions = this;

    scroll.on("active", function (index) {
      console.log(scroll);
      d3.selectAll(".step")
        .transition()
        .duration(500)
        .style("opacity", function (d, i) {
          console.log("i is " + index);
          return i === index ? 1 : 0.1;
        });

      activeIndex = index;
      scroll_functions.active_index = activeIndex;

      let sign = activeIndex - lastIndex < 0 ? -1 : 1;
      let scrolledSections = d3.range(
        lastIndex + sign,
        activeIndex + sign,
        sign
      );
      scrolledSections.forEach((i) => {
        console.log("section " + i);
        if (i == 0) {
          scroll_functions.hideMap();
          scroll_functions.hideBubbles();
        }
        if (i == 1) {
          scroll_functions.drawMap();
          scroll_functions.hideLegend();
          scroll_functions.hideRoute12();
        }
        if (i == 2) {
          scroll_functions.hideRoute3();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorCentral();
        } else if (i == 3) {
          scroll_functions.hideRoute12();
          scroll_functions.resetZoom();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorWestern();
        } else if (i == 4) {
          scroll_functions.hideRoute3();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorWestern2();
          scroll_functions.hideRoute6();
        } else if (i == 5) {
          scroll_functions.hideRoute45();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorEastern();
          scroll_functions.hideBubbles();
        }
      });
      lastIndex = activeIndex;
    });

    scroll.on("progress", function (index, progress) {
      if ((index == 2) & (progress > 0.7)) {
      }
    });
  },
  methods: {
    scroller: function () {
      let container = d3.select("body");
      let dispatch = d3.dispatch("active", "progress");
      let sections = d3.selectAll(".step");
      let sectionPositions;
      var self = this;

      let currentIndex = -1;
      let containerStart = 0;

      const scroll = () => {
        d3.select(window)
          .on("scroll.scroller", position)
          .on("resize.scroller", resize);

        resize();

        console.log(this);

        let timer = d3.timer(function () {
          position();
          timer.stop();
        });
      };

      function resize() {
        sectionPositions = [];
        let startPos;

        sections.each(function (d, i) {
          let top = this.getBoundingClientRect().top;

          if (i === 0) {
            startPos = top;
          }
          sectionPositions.push(top - startPos);
        });
      }

      function position() {
        let pos = window.pageYOffset - 400 - containerStart;
        let sectionIndex = d3.bisect(sectionPositions, pos);
        sectionIndex = Math.min(sections.size() - 1, sectionIndex);

        if (currentIndex !== sectionIndex) {
          self.state = "active";
          dispatch.call("active", this, sectionIndex);
          currentIndex = sectionIndex;
          console.log(self.state);
        }

        let prevIndex = Math.max(sectionIndex - 1, 0);
        let prevTop = sectionPositions[prevIndex];
        let progress =
          (pos - prevTop) / (sectionPositions[sectionIndex] - prevTop);
        dispatch.call("progress", this, currentIndex, progress);
        self.state = "progress";
        console.log(self.state);
      }

      scroll.container = function (value) {
        if (arguments.legth === 0) {
          return container;
        }
        container = value;
        return scroll;
      };

      scroll.on = function (action, callback) {
        dispatch.on(action, callback);
      };

      return scroll;
    },

    drawInitial: function () {
      this.svg = d3
        .select("#map")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);

      const projection = d3
        .geoMercator()
        .fitSize([this.width, this.height], map_json);

      console.log();

      const path = d3.geoPath().projection(projection);
      this.svg
        .append("g")
        .selectAll(".country")
        .data(map_json.features)
        .enter()
        .append("path")
        .attr("d", path)
        .attr("class", "country")
        .attr("fill", "#c4c1b6")
        .attr("stroke", "black");

      d3.selectAll(".country").attr("opacity", 0);

      //drawing legend

      var color = d3
        .scaleOrdinal()
        .domain(this.legend_keys)
        .range(["#800020", "#006b80"]);

      this.svg
        .selectAll(".mydots")
        .data(this.legend_keys)
        .enter()
        .append("circle")
        .attr("class", "mydots")
        .attr("cx", 1300)
        .attr("cy", function (d, i) {
          return 100 + i * 25;
        })
        .attr("r", 10)
        .attr("opacity", 0)
        .style("fill", function (d) {
          return color(d);
        });

      // Add one dot in the legend for each name.
      this.svg
        .selectAll(".mylabels")
        .data(this.legend_keys)
        .enter()
        .append("text")
        .attr("class", "mylabels")
        .attr("x", 1320)
        .attr("y", function (d, i) {
          return 100 + i * 25;
        })
        .style("font-size", "30px")
        .style("fill", function (d) {
          return color(d);
        })
        .text(function (d) {
          return d;
        })
        .attr("text-anchor", "left")
        .attr("opacity", 0)
        .style("alignment-baseline", "middle");

      let tunis = [10, 33];
      let tunis_mid = [10.6, 36.41];
      let libya = [13, 25];
      let libya_mid = [16, 30];
      let italy = [15, 41];
      let mor = [-13.5, 25.5];
      let mor_mid = [-15, 29];
      let spain = [-5, 39];
      let Mauritania = [-13.4357, 15];
      let maur_mid = [-19.13, 16.5];
      let senegal_mid = [-20.71, 13.59];
      let senegal = [-11.51, 11];
      let canary = [-16.16, 28.2916];
      let turkey = [30, 40];
      let turkey_mid = [28, 37];
      let greece = [22, 37.655];

      const curve = d3.line().curve(d3.curveNatural);

      const pointsLine1 = [
        projection(tunis),
        projection(tunis_mid),
        projection(italy),
      ];
      const pointsLine2 = [
        projection(libya),
        projection(libya_mid),
        projection(italy),
      ];
      const pointsLine3 = [
        projection(mor),
        projection(mor_mid),
        projection(spain),
      ];
      const pointsLine5 = [
        projection(senegal),
        projection(senegal_mid),
        projection(canary),
      ];
      const pointsLine4 = [
        projection(Mauritania),
        projection(maur_mid),
        projection(canary),
      ];
      const pointsLine6 = [
        projection(turkey),
        projection(turkey_mid),
        projection(greece),
      ];
      this.svg
        .selectAll(".route1")
        .data([pointsLine1])
        .join("path")
        .attr("class", "route1")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");

      this.svg
        .selectAll(".route2")
        .data([pointsLine2])
        .join("path")
        .attr("class", "route2")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");

      this.svg
        .selectAll(".route3")
        .data([pointsLine3])
        .join("path")
        .attr("class", "route3")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");

      this.svg
        .selectAll(".route4")
        .data([pointsLine4])
        .join("path")
        .attr("class", "route4")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");
      this.svg
        .selectAll(".route5")
        .data([pointsLine5])
        .join("path")
        .attr("class", "route5")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");

      this.svg
        .selectAll(".route6")
        .data([pointsLine6])
        .join("path")
        .attr("class", "route6")
        .attr("d", curve)
        .attr("stroke", "#766F81")
        .attr("stroke-width", "5px")
        .attr("fill", "none")
        .attr("opacity", "0");
    },

    hideMap: function () {
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Turkey" || d.properties.name === "Greece"
        )
        .transition()
        .duration(1500)
        .attr("fill", "#c4c1b6");

      var totalLength = d3.selectAll(".country").node().getTotalLength();
      d3.selectAll(".country")
        .transition()
        .duration(1000)
        .attr("fill", "none")
        .attr("stroke-dasharray", totalLength + " " + totalLength)
        .attr("stroke-dashoffset", -totalLength)
        .transition()
        .duration(1500)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", -totalLength)
        .attr("opacity", 0);

      d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 0);
    },
    drawMap: function () {
      d3.selectAll(".country").attr("opacity", 1);
      var totalLength = d3.selectAll(".country").node().getTotalLength();

      d3.selectAll(".country")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength + " " + totalLength)
        .attr("stroke-dashoffset", totalLength)
        .transition()
        .duration(800)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0)
        .attr("fill", "#c4c1b6");
    },
    drawMapNoTransition: function () {
      d3.selectAll(".country").attr("opacity", 1);

      d3.selectAll(".country")
        .attr("opacity", 1)
        .transition()
        .attr("fill", "#c4c1b6");
    },
    colorCentral: function () {
      const projection = d3
        .geoMercator()
        .fitSize([this.width, this.height], map_json);

      const totalLength_route1 = d3
        .selectAll(".route1")
        .node()
        .getTotalLength();
      const totalLength_route2 = d3
        .selectAll(".route2")
        .node()
        .getTotalLength();

      d3.selectAll(".route1")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route1 + " " + totalLength_route1)
        .attr("stroke-dashoffset", totalLength_route1)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll(".route2")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route2 + " " + totalLength_route2)
        .attr("stroke-dashoffset", totalLength_route2)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomLines1 = d3.zoom().on("zoom", zoomedLines1);
      var zoomLines2 = d3.zoom().on("zoom", zoomedLines2);
      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);
      var zoomLines4 = d3.zoom().on("zoom", zoomedLines4);
      var zoomLines5 = d3.zoom().on("zoom", zoomedLines5);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);
      let that = this;
      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }

      function zoomedLines1() {
        d3.selectAll(".route1").attr("transform", d3.event.transform);
      }

      function zoomedLines2() {
        d3.selectAll(".route2").attr("transform", d3.event.transform);
      }

      function zoomedLines3() {
        d3.selectAll(".route3").attr("transform", d3.event.transform);
      }

      function zoomedLines4() {
        d3.selectAll(".route4").attr("transform", d3.event.transform);
      }

      function zoomedLines5() {
        d3.selectAll(".route5").attr("transform", d3.event.transform);
      }

      function zoomedLines6() {
        d3.selectAll(".route6").attr("transform", d3.event.transform);
      }

      d3.selectAll(".country")
        .transition()
        .delay(1000)
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-600, -100).scale(1.8));

      d3.selectAll(".route1")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines1.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );
      d3.selectAll(".route2")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines2.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );
      d3.selectAll(".route3")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines3.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );

      d3.selectAll(".route4")
        .transition()
        .duration(750)
        .call(
          zoomLines4.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );

      d3.selectAll(".route5")
        .transition()
        .duration(750)
        .call(
          zoomLines5.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );

      d3.selectAll(".route6")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines6.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );
    },
    colorWestern: function () {
      const totalLength_route3 = d3
        .selectAll(".route3")
        .node()
        .getTotalLength();

      d3.selectAll(".route3")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route3 + " " + totalLength_route3)
        .attr("stroke-dashoffset", totalLength_route3)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }

      d3.selectAll(".country")
        .transition()
        .delay(1000)
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-400, -400).scale(1.7));

      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);

      function zoomedLines3() {
        d3.selectAll(".route3").attr("transform", d3.event.transform);
      }

      d3.selectAll(".route3")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines3.transform,
          d3.zoomIdentity.translate(-400, -400).scale(1.7)
        );

      var zoomLines4 = d3.zoom().on("zoom", zoomedLines4);
      var zoomLines5 = d3.zoom().on("zoom", zoomedLines5);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);

      function zoomedLines4() {
        d3.selectAll(".route4").attr("transform", d3.event.transform);
      }

      function zoomedLines5() {
        d3.selectAll(".route5").attr("transform", d3.event.transform);
      }

      function zoomedLines6() {
        d3.selectAll(".route6").attr("transform", d3.event.transform);
      }

      d3.selectAll(".route4")
        .transition()
        .duration(750)
        .call(
          zoomLines4.transform,
          d3.zoomIdentity.translate(-400, -400).scale(1.7)
        );

      d3.selectAll(".route5")
        .transition()
        .duration(750)
        .call(
          zoomLines5.transform,
          d3.zoomIdentity.translate(-400, -400).scale(1.7)
        );

      d3.selectAll(".route6")
        .transition()

        .duration(750)
        .call(
          zoomLines6.transform,
          d3.zoomIdentity.translate(-400, -400).scale(1.7)
        );

      const totalLength_route1 = d3
        .selectAll(".route1")
        .node()
        .getTotalLength();
      const totalLength_route2 = d3
        .selectAll(".route2")
        .node()
        .getTotalLength();

      d3.selectAll(".route1")
        .attr("opacity", 0)
        .attr("stroke-dasharray", totalLength_route1 + " " + totalLength_route1)
        .attr("stroke-dashoffset", totalLength_route1)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll(".route2")
        .attr("opacity", 0)
        .attr("stroke-dasharray", totalLength_route2 + " " + totalLength_route2)
        .attr("stroke-dashoffset", totalLength_route2)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);
    },
    colorWestern2: function () {
      const totalLength_route4 = d3
        .selectAll(".route4")
        .node()
        .getTotalLength();
      const totalLength_route5 = d3
        .selectAll(".route5")
        .node()
        .getTotalLength();

      d3.selectAll(".route4")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route4 + " " + totalLength_route4)
        .attr("stroke-dashoffset", totalLength_route4)
        .transition()
        .delay(750)
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll(".route5")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route5 + " " + totalLength_route5)
        .attr("stroke-dashoffset", totalLength_route5)
        .transition()
        .delay(750)
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);
    },
    colorEastern: function () {
      const totalLength_route6 = d3
        .selectAll(".route6")
        .node()
        .getTotalLength();

      d3.selectAll(".route6")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route6 + " " + totalLength_route6)
        .attr("stroke-dashoffset", totalLength_route6)
        .transition()
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }
      function zoomedLines6() {
        d3.selectAll(".route6").attr("transform", d3.event.transform);
      }

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);

      d3.selectAll(".country")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity.translate(-1000, -200).scale(1.9)
        );

      console.log("route 6 is ", d3.selectAll(".route6"));
      d3.selectAll(".route6")
        .transition()
        .delay(1000)
        .duration(750)
        .call(
          zoomLines6.transform,
          d3.zoomIdentity.translate(-1000, -200).scale(1.9)
        );
    },
  },
};
</script>

<style scoped>
#sections {
  position: relative;
  /* display: inline-block; */
  width: 40rem;
  /* top: 60px; */
  z-index: 90;
  margin-left: 1rem;
  /* height: 10000px; */
}

.step {
  margin-bottom: 1rem;
  height: 50rem;
  /* font-family: "Domine"; */
  font-family: "Courier New", Courier, monospace;
  font-weight: 400;
  line-height: 1.4em;
  font-size: 1.5em;
  text-align: justify;
  margin-left: 0.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

#map {
  /* display: inline-block; */
  position: fixed;
  top: 1rem;
  right: -50rem;
  /* right: -750px; */
  z-index: 1;
  /* margin-left: 0; */
  /* height: 100rem; */
  /* width: 1000rem; */
}
</style>