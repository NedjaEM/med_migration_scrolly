<template>
  <div id="app">
    <!-- <MapIntro></MapIntro> -->
    <div id="graphic">
      <div id="sections">
        <Story :state="state" :active_index="active_index"></Story>
      </div>
      <div id="map"></div>
      <!-- <div id="photos"></div> -->
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import MapIntro from "./components/MapIntro.vue";
import Story from "./components/Story.vue";
import * as d3 from "../../lib/d3";
import map_json from "../public/Data/map.geo.json";
import bar_data from "../public/Data/aggregate_data.csv";

export default {
  name: "App",
  components: {
    MapIntro,
    Story,
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
    console.log(bar_data);
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
          scroll_functions.hideLegend()
        }
        if (i == 2) {
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorCentral();
        } else if (i == 3) {
          scroll_functions.drawMapNoTransition()
          scroll_functions.colorWestern();
        } else if (i == 4) {
          scroll_functions.drawMapNoTransition()
          scroll_functions.colorWestern2();
        } else if (i == 5) {
          scroll_functions.drawMapNoTransition()
          scroll_functions.colorEastern();
          scroll_functions.hideBubbles()
        } else if (i == 6) {
          scroll_functions.hideMap();
          scroll_functions.drawBarChart();
        } else if (i == 7) {
          scroll_functions.chart2014();
 
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
      console.log("val " + this.svg);
      // const width = window.innerWidth * 1.2,
      //   height = window.innerHeight * 1.4,
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
        .selectAll(".country")
        .data(map_json.features)
        .join("path")
        .attr("d", path)
        .attr("class", "country")
        .attr("fill", "transparent")
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

      d3.csv(bar_data, d3.autoType).then((data) => {
        this.width = window.innerWidth;
        this.height = window.innerHeight;
        var x = d3
          .scaleLinear()
          .domain([2014, 2020])
          .range([this.width / 5, this.width / 1.2]);

        var y = d3
          .scaleLinear()
          .domain([2014, 2020])
          .range([this.height / 1.5, this.height / 7]);

        var z = d3
          .scaleLinear()
          .domain([0, d3.max(data, (d) => d["Total Arrivals"])])
          .range([40, 120]);

        this.svg
          .append("g")
          .selectAll(".dot")
          
          .data(data)
          .enter()
          .append("circle")
          .attr("class", "dot")
          .attr("cx", function (d) {
            return x(d.Year);
          })
          .attr("cy", function (d) {
            return y(d.Year);
          })
          .attr("r", function (d) {
            return z(d["Total Arrivals"]);
          })
          .style("fill", "#53292a")
          .style("fill-opacity",0.5)
          .style("opacity", 0)
          .attr("stroke", "black")
          .append("text")
          .attr("dx", function(d){return x(d.Year)})
          .attr("dy", function(d){return y(d.Year)})
          .text(function(d){return d.Year});

      });
    },

    hideMap: function () {
      d3.selectAll(".country")
        .transition()
        .duration(1000)
        .attr("fill", "none")
        .attr("opacity", 0);

             d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 0);
      
    },

    hideLegend: function () {
      d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 0);
    },

    hideBubbles: function () {
      d3.selectAll(".dot")
      .transition().duration(1000)
      .style("opacity", 0);
    },

    drawMap: function () {
      d3.selectAll(".country").attr("opacity", 1);
      var totalLength = d3.selectAll(".country").node().getTotalLength();

      d3.selectAll(".country")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength + " " + totalLength)
        .attr("stroke-dashoffset", totalLength)
        .transition()
        .duration(1500)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0)
        .attr("fill-opacity", 0);
     
    },
     drawMapNoTransition: function () {
      d3.selectAll(".country").attr("opacity", 1);
     

      d3.selectAll(".country")
        .attr("opacity", 1)
        .transition()
        .attr("fill-opacity", 0);
     
    },
    colorCentral: function () {
      console.log("coloring central");
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Tunisia" || d.properties.name === "Libya"
        )
        .transition()
        .duration(1500)
        .attr("fill", "#800020")
        .attr("fill-opacity", "0.5");

      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Italy")
        .transition()
        .duration(1500)
        .attr("fill", "#006b80")
        .attr("fill-opacity", "0.5");
      console.log(d3.select("mylabels"));

      d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 1);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 1);
    },
    colorWestern: function () {
      console.log("coloring Western");
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Tunisia" ||
            d.properties.name === "Libya" ||
            d.properties.name === "Italy"
        )
        .transition()
        .duration(1500)
        .attr("fill", "none");

      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Morocco")
        .transition()
        .duration(1500)
        .attr("fill", "#800020")
        .attr("fill-opacity", "0.5");
      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Spain")
        .transition()
        .duration(1500)
        .attr("fill", "#006b80")
        .attr("fill-opacity", "0.5");
    },
    colorWestern2: function () {
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Morocco" || d.properties.name === "Spain"
        )
        .transition()
        .duration(1500)
        .attr("fill", "grey");

      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Mauritania" ||
            d.properties.name === "Senegal"
        )
        .transition()
        .duration(1500)
        .attr("fill", "#800020")
        .attr("fill-opacity", "0.5");
      d3.selectAll(".country")
        .filter((d) => d.properties.name === "the Canary Islands")
        .transition()
        .duration(1500)
        .attr("fill", "#006b80")
        .attr("fill-opacity", "0.5");
    },
    colorEastern: function () {
      console.log("coloring central");
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Morocco" || d.properties.name === "Spain"
        )
        .transition()
        .duration(1500)
        .attr("fill", "none");
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Mauritania" ||
            d.properties.name === "Senegal"
        )
        .transition()
        .duration(1500)
        .attr("fill", "none");
      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Turkey")
        .transition()
        .duration(1500)
        .attr("fill", "#800020")
        .attr("fill-opacity", "0.5");
      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Greece")
        .transition()
        .duration(1500)
        .attr("fill", "#006b80")
        .attr("fill-opacity", "0.5");
      // .filter())
    },
    firstImage: function () {
      console.log("calling");
      var img = d3
        .select("#photos")
        .append("img")
        .attr(
          "src",
          "https://api.time.com/wp-content/uploads/2015/10/italy-migrants-refugees-asylum-seekers-1.jpg"
        )
        .style("opacity", 0);
      // .attr("right","-500px")
      console.log(img);
      img.transition().duration(4000).ease(d3.easeLinear).style("opacity", 1);
    },

    drawBarChart: function () {
      d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".dot")
      .transition().duration(1000)
      .style("opacity", 0.7);
    },

    chart2014: function () {
      console.log(d3.selectAll(".dot"))
      d3.selectAll(".dot")
        .filter(
          (d) =>
            d.Year = 2014
        )
        .transition()
        .duration(1000)
        .style("opacity",0)
    }
  },
};
</script>

<style>
#sections {
  position: relative;
  /* display: inline-block; */
  width: 40rem;
  /* top: 60px; */
  z-index: 90;
  margin-left: 1rem;
  /* height: 10000px; */
}

@media (max-width: 900px) {
  #sections {
    position: relative;
    /* display: inline-block; */
    width: 10rem;
    /* top: 60px; */
    z-index: 90;
    /* margin-left: rem; */
  }
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

p, text {
  font-family: "Courier New", Courier, monospace;
  /* font-weight: 800; */
  line-height: 1.4em;
  font-size: 1em;
}

@media (max-width: 900px) {
  .step {
    margin-bottom: 1rem;
    height: 50rem;
    /* font-family: "Domine"; */
    font-family: "Courier New", Courier, monospace;
    font-weight: 400;
    line-height: 1.4em;
    font-size: 0.5em;
    text-align: justify;
    /* margin-left: 0.5rem; */
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }
}
#graphic {
  /* margin: auto; */
  width: 100rem;
  flex-direction: row;
  align-items: top;
  justify-content: space-around;
}

@media (max-width: 900px) {
  #graphic {
    width: 100rem;
    flex-direction: row;
    align-items: top;
    justify-content: space-around;
  }
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

#photos {
  /* display: inline-block; */
  position: fixed;
  top: 1rem;

  left: 500px;
  z-index: 1;
  /* margin-left: 0; */
  height: 10px;
  width: 100px;
}

/* 
rect{
  z-index: 100;
} */

#app {
  /* font-family: Helvetica, Arial, sans-serif; */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: #dbd9d2;
  margin-top: 1rem;
  /* max-width: 100rem; */
}
</style>
