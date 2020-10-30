<template>
  <div id="app">
    <MapIntro></MapIntro>
    <div id="graphic">
      <div id="sections">
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>
            Europe's Mediterrenean Border is known to be the most lethal border
            in the world
          </h1>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>There are multiple routes that migrants follow to get to Europe through the Mediterrenean</h1>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>Central Route</h1>
          <p>Which starts in North Africa (mainly Libya and Tunisia) to Italy</p>
          <p>The closest island Lampedusa is reachable in 1 day from Tunisia and three days from 
            Libya </p>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>Western Route</h1>
          <p>From Morocco to Spain, spanning only 15 Km, making it the shortest route.</p>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <p>After the Morocco-Spain Western route was closed, the route from Mauritania and Senegal to 
            the Canary Islands became a more popular Western route.
          </p>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>Eastern Route</h1>
          <p>From Turkey to Greece. This route became prominent between 2014 and 2016 following the political
            situation in Syria.
          </p>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>Let's look into the numbers of arrivals through the Mediterrenean to Europe </h1>
          <p>
          </p>
        </section>
        <section class="step">
          <div class="title"></div>
          <br /><br /><br /><br />
          <h1>Let's look into the numbers of arrivals through the Mediterrenean to Europe </h1>
          <p>
          </p>
        </section>
      </div>
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import MapIntro from "./components/MapIntro.vue";
import * as d3 from "../../lib/d3";
import map_json from "../public/Data/map.geo.json";

export default {
  name: "App",
  components: {
    MapIntro,
  },
  mounted() {
    this.drawInitial();
    let scroll = this.scroller().container(d3.select("#graphic"));
    scroll();

    console.log(map_json)

    // let activationFunctions = [
    //   this.drawMap
    //   ];

    // console.log(activationFunctions)

 
  let lastIndex,
      activeIndex = 0;


    var scroll_functions = this;

    scroll.on("active", function (index) {
      console.log(scroll)
      d3.selectAll(".step")
        .transition()
        .duration(500)
        .style("opacity", function (d, i) {
          return i === index ? 1 : 0.1;
        });

      activeIndex = index;
      let sign = activeIndex - lastIndex < 0 ? -1 : 1;
      let scrolledSections = d3.range(
        lastIndex + sign,
        activeIndex + sign,
        sign
      );
      scrolledSections.forEach((i) => {
        console.log("section "+ i)
        if (i==0) {
          scroll_functions.hideMap()
        }
        if (i==1){
          scroll_functions.drawMap()
        }
        if (i==2) {
          scroll_functions.colorCentral()
        }
        else if (i==3){
          scroll_functions.colorWestern()
        }
        else if (i==4){
          scroll_functions.colorWestern2()
        }
        else if (i==5){
          scroll_functions.colorEastern()
        }
        else if (i==6){
           scroll_functions.hideMap()
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

      let currentIndex = -1;
      let containerStart = 0;

      function scroll() {
        d3.select(window)
          .on("scroll.scroller", position)
          .on("resize.scroller", resize);

        resize();

        let timer = d3.timer(function () {
          position();
          timer.stop();
        });
      }

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
        let pos = window.pageYOffset - 800 - containerStart;
        let sectionIndex = d3.bisect(sectionPositions, pos);
        sectionIndex = Math.min(sections.size() - 1, sectionIndex);

        if (currentIndex !== sectionIndex) {
          dispatch.call("active", this, sectionIndex);
          currentIndex = sectionIndex;
        }

        let prevIndex = Math.max(sectionIndex - 1, 0);
        let prevTop = sectionPositions[prevIndex];
        let progress =
          (pos - prevTop) / (sectionPositions[sectionIndex] - prevTop);
        dispatch.call("progress", this, currentIndex, progress);
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
      const width = window.innerWidth * 1.2,
        height = window.innerHeight * 1.4,
        svg = d3
          .select("#map")
          .append("svg")
          .attr("width", width)
          .attr("height", height);

      console.log(width);
      const projection = d3.geoMercator().fitSize([width, height], map_json);

      console.log();

      const path = d3.geoPath().projection(projection);
      svg
        .selectAll(".country")
        .data(map_json.features)
        .join("path")
        .attr("d", path)
        .attr("class", "country")
        .attr("fill", "transparent")
        .attr("stroke", "black");

      d3.selectAll(".country").attr("opacity", 0);
    },

     hideMap: function(){
       d3.selectAll(".country")
        .transition()
        .duration(1000)
        .attr("fill","none")
        .attr("opacity", 0)
        
     },
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
            .duration(2000)
            .ease(d3.easeLinear)
            .attr("stroke-dashoffset", 0)
    },
    colorCentral: function() {
      console.log("coloring central")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Tunisia" || d.properties.name === "Libya")
      .transition()
      .duration(1500)
      .attr("fill","#800020")
      .attr("fill-opacity", "0.5")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Italy")
      .transition()
      .duration(1500)
      .attr("fill", "#006b80")
      .attr("fill-opacity", "0.5")
            // .filter())
    },
    colorWestern: function() {
      console.log("coloring Western")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Tunisia" || d.properties.name === "Libya" || d.properties.name === "Italy")
      .transition()
      .duration(1500)
      .attr("fill","none")
    
       d3.selectAll(".country")
      .filter(d => d.properties.name === "Morocco" )
      .transition()
      .duration(1500)
      .attr("fill","#800020")
      .attr("fill-opacity", "0.5")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Spain")
      .transition()
      .duration(1500)
      .attr("fill", "#006b80")
      .attr("fill-opacity", "0.5")
      
    },
    colorWestern2: function() {
       d3.selectAll(".country")
      .filter(d => d.properties.name === "Morocco" || d.properties.name === "Spain" )
      .transition()
      .duration(1500)
      .attr("fill","grey")
 
       d3.selectAll(".country")
      .filter(d => d.properties.name === "Mauritania" || d.properties.name === "Senegal" )
      .transition()
      .duration(1500)
      .attr("fill","#800020")
      .attr("fill-opacity", "0.5")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "the Canary Islands")
      .transition()
      .duration(1500)
      .attr("fill", "#006b80")
      .attr("fill-opacity", "0.5")
    },
    colorEastern: function() {
      console.log("coloring central")
       d3.selectAll(".country")
      .filter(d => d.properties.name === "Morocco" || d.properties.name === "Spain" )
      .transition()
      .duration(1500)
      .attr("fill","none")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Mauritania" || d.properties.name === "Senegal")
      .transition()
      .duration(1500)
      .attr("fill","none")
       d3.selectAll(".country")
      .filter(d => d.properties.name === "Turkey" )
      .transition()
      .duration(1500)
      .attr("fill","#800020")
      .attr("fill-opacity", "0.5")
      d3.selectAll(".country")
      .filter(d => d.properties.name === "Greece")
      .transition()
      .duration(1500)
      .attr("fill", "#006b80")
      .attr("fill-opacity", "0.5")
            // .filter())
    },
    firstImage: function(){

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

p {
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
  height: 1000rem;
  /* width: 1000rem; */
}




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
