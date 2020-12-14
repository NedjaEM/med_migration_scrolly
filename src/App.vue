<template>
  <main class="scroll-container">
    <VueScrollProgress></VueScrollProgress>
    <header class="header">
      <h1 class="header_title">
        Mapping State Violence
        <br />
        and Colonial Legacies in the Mediterranean
      </h1>
      <nav>
        <ul class="v">
          <button
            class="header_navItem open-modal1"
            type="button"
            data-open="modal1"
          >
            <a href="#">About</a>
          </button>
          <button
            class="header_navItem open-modal2"
            type="button"
            data-open="modal1"
          >
            <a href="#">Contact</a>
          </button>
        </ul>
      </nav>
    </header>

    <div id="app">
      <section class="landing">
        <div class="banksy"></div>
        <div class="intro">
          <h2>
            Migrants and State Violence <br />
            in the Mediterranean
          </h2>
          <h3>
            Europe's Mediterrenean Border is known to be the most lethal border
            in the world. <br />
            <br />
            This project aims to highlight the Mediterranean migration events
            between 2014 and 2019 <br />
            by unfolding the involvment of nation states in the crisis.
          </h3>
          <br />
          <br />
          <br />
          <div class="content">
            <svg id="more-arrows">
              <polygon
                class="arrow-bottom"
                fill="#898883"
                points="37.6,64 0,36.1 5.1,32.8 37.6,56.8 70.4,32.8 75.5,36.1 "
              />
            </svg>
          </div>
        </div>
      </section>
      <!-- <MapIntro></MapIntro> -->
      <div id="graphic">
        <div id="sections">
          <Story :state="state" :active_index="active_index"></Story>
        </div>
        <div id="map"></div>
        <!-- <div id="photos"></div> -->
      </div>
      <div id="graphic2">
        <div id="sections">
          <Story2 :state="state" :active_index="active_index"></Story2>
        </div>
        <div id="bub">
          <div class="year">the year is</div>
        </div>
      </div>
      <div id="graphic3">
        <div id="sections">
          <Story3 :state="state" :active_index="active_index"></Story3>
        </div>
        <div id="bub2"></div>
      </div>
    </div>
  </main>
</template>

<script>
/* eslint-disable */
import MapIntro from "./components/MapIntro.vue";
import Story from "./components/Story.vue";
import Story2 from "./components/Story2.vue";
import Story3 from "./components/Story3.vue";
import * as d3 from "../../lib/d3";
import map_json from "../public/Data/map.geo.json";
import bar_data from "../public/Data/aggregate_data.csv";
import missing_data from "../public/Data/med_migrants.csv";
import VueScrollProgress from "vue-scroll-progress";
import Vue from "vue";

Vue.use(VueScrollProgress);

export default {
  name: "App",
  components: {
    MapIntro,
    Story,
    Story2,
    Story3,
    VueScrollProgress,
  },
  data: function () {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
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
      svg2: d3
        .select("#bub")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height),
    };
  },
  mounted() {
    this.drawInitial();
    this.drawInitialBub();

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
          return i === index ? 0.1 : 1;
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
        console.log("function ", i);
        console.log("section ", i);
        if (i == 0) {
          scroll_functions.hideMap();
          scroll_functions.removeHighlight();
        }
        if (i == 1) {
          scroll_functions.drawMap();
          scroll_functions.hideRoute12();
        }
        if (i == 2) {
          scroll_functions.hideRoute3();
          scroll_functions.colorCentral();
        } else if (i == 3) {
          scroll_functions.hideRoute12();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorWestern();
          scroll_functions.hideRoute45();
        } else if (i == 4) {
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorWestern2();
          scroll_functions.hideRoute6();
          scroll_functions.hideRoute3();
        } else if (i == 5) {
          scroll_functions.hideRoute45();
          scroll_functions.drawMapNoTransition();
          scroll_functions.colorEastern();
          scroll_functions.hideChart();
          d3.selectAll(".legend").transition().style("opacity", 0);
        } else if (i == 6) {
          scroll_functions.removeHighlight();
          scroll_functions.hideMap();
          scroll_functions.hideRoute6();
          scroll_functions.showIcons();
        } else if (i == 7) {
          scroll_functions.highlight2014();
        } else if (i == 10) {
          scroll_functions.highlight2015();
        } else if (i == 13) {
          scroll_functions.highlight2016();
        } else if (i == 16) {
          scroll_functions.highlight2017();
        } else if (i == 17) {
          scroll_functions.highlight2018();
        } else if (i == 18) {
          scroll_functions.highlight2019();
        } else if (i == 19) {
          scroll_functions.removeHighlight();
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

    drawInitialBub: function () {
      this.svg = d3
        .select("#bub")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);

      const projection = d3
        .geoMercator()
        .fitSize([2.4 * this.width, 2.4 * this.height], map_json);

      const path = d3.geoPath().projection(projection);

      this.svg
        .append("defs")
        .append("g")
        .attr("id", "iconCustom")
        .append("path")
        .attr(
          "d",
          "M3.5,2H2.7C3,1.8,3.3,1.5,3.3,1.1c0-0.6-0.4-1-1-1c-0.6,0-1,0.4-1,1c0,0.4,0.2,0.7,0.6,0.9H1.1C0.7,2,0.4,2.3,0.4,2.6v1.9c0,0.3,0.3,0.6,0.6,0.6h0.2c0,0,0,0.1,0,0.1v1.9c0,0.3,0.2,0.6,0.3,0.6h1.3c0.2,0,0.3-0.3,0.3-0.6V5.3c0,0,0-0.1,0-0.1h0.2c0.3,0,0.6-0.3,0.6-0.6V2.6C4.1,2.3,3.8,2,3.5,2z"
        );

      //specify the number of columns and rows for pictogram layout
      var numCols = 20;
      var numRows = 20;

      //padding for the grid
      var xPadding = 30;
      var yPadding = 30;

      //horizontal and vertical spacing between the icons
      var hBuffer = 8;
      var wBuffer = 8;

      //generate a d3 range for the total number of required elements
      var myIndex = d3.range(numCols * numRows);

      //text element to display number of icons highlighted
      this.svg
        .append("text")
        .attr("id", "txtValue1")
        .attr("x", 0.46 * this.width - 200)
        .attr("y", 0.13 * this.height)
        .attr("dy", -3)
        .attr("class", "icon-text1")
        .text("1,927,007")
        .attr("opacity", "0");

      this.svg
        .append("text")
        .attr("id", "txtValue")
        .attr("x", 0.46 * this.width - 200)
        .attr("y", 0.16 * this.height)
        .attr("dy", -3)
        .attr("class", "icon-text2")
        .text(" people arrived to Europe through")
        .attr("opacity", "0");

      this.svg
        .append("text")
        .attr("id", "txtValue")
        .attr("x", 0.46 * this.width - 200)
        .attr("y", 0.18 * this.height)
        .attr("dy", -3)
        .attr("class", "icon-text2")
        .text("the Mediterranean between 2014 and 2019")
        .attr("opacity", "0");

      //create group element and create an svg <use> element for each icon
      this.svg
        .append("g")
        .attr("id", "pictoLayer")
        .selectAll(".iconPlain")
        .data(myIndex)
        .enter()
        .append("use")
        .attr("xlink:href", "#iconCustom")
        .attr("id", function (d) {
          return "icon" + d;
        })
        .attr("x", function (d) {
          var remainder = d % numCols;
          return xPadding + remainder * wBuffer - 70;
          console.log(x);
        })
        .attr("y", function (d) {
          var whole = Math.floor(d / numCols);
          return yPadding + whole * hBuffer;
        })

        .classed("iconPlain", true)
        .attr("class", "iconPlain")
        .attr("opacity", "0");

      this.svg
        .append("defs1")
        .append("g")
        .attr("id", "iconCustomLegend")
        .append("path")
        .attr(
          "d",
          "M3.5,2H2.7C3,1.8,3.3,1.5,3.3,1.1c0-0.6-0.4-1-1-1c-0.6,0-1,0.4-1,1c0,0.4,0.2,0.7,0.6,0.9H1.1C0.7,2,0.4,2.3,0.4,2.6v1.9c0,0.3,0.3,0.6,0.6,0.6h0.2c0,0,0,0.1,0,0.1v1.9c0,0.3,0.2,0.6,0.3,0.6h1.3c0.2,0,0.3-0.3,0.3-0.6V5.3c0,0,0-0.1,0-0.1h0.2c0.3,0,0.6-0.3,0.6-0.6V2.6C4.1,2.3,3.8,2,3.5,2z"
        );

      var myIndex_legend = d3.range(1);

      this.svg
        .append("g")
        .attr("id", "legend_year")
        .attr("class", "legend_year")
        .selectAll("div")
        .data(myIndex_legend)
        .enter()
        .append("use")
        .attr("xlink:href", "#iconCustomLegend")
        .attr("x", 0)
        .attr("y", 0);

      var zoomLegend = d3.zoom().on("zoom", zoomedLegend);

      function zoomedLegend() {
        d3.selectAll(".legend_year").attr("transform", d3.event.transform);
      }

      d3.selectAll(".legend_year")
        .transition()
        .duration(750)
        .call(
          zoomLegend.transform,
          d3.zoomIdentity
            .translate(0.4 * this.width, 0.9 * this.height)
            .scale(this.width / 500)
        );

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", 0.42 * this.width + 10)
        .attr("y", 0.9 * this.height + 15)
        .attr("class", "legend")
        .text("4,817 migrants")
        .attr("opacity", "0");

      d3.selectAll(".legend_year").attr("opacity", 0);

      let x1 = d3.select("#icon399").attr("x");
      let y1 = d3.select("#icon399").attr("y");

      let x2 = d3.select("#icon250").attr("x");
      let y2 = d3.select("#icon250").attr("y");

      console.log("xxxx ", d3.selectAll("#icon399").attr("y"));
      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 9)
        .attr("class", "picto1")
        .text("209,660 migrants")
        .attr("opacity", "0");

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 5 - 10)
        .attr("class", "picto2")
        .text("1,017,294 migrants")
        .attr("opacity", "0");

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 1.8 - 10)
        .attr("class", "picto3")
        .text("356,502 migrants")
        .attr("opacity", "0");

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 1.5 - 10)
        .attr("class", "picto4")
        .text("171,050 migrants")
        .attr("opacity", "0");

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 1.35 - 10)
        .attr("class", "picto5")
        .text("114,775 migrants")
        .attr("opacity", "0");

      this.svg
        .selectAll("#pictoLayer")
        .append("text")
        .attr("x", this.width - this.width / 2.8)
        .attr("y", this.height - this.height / 1.29 - 10)
        .attr("class", "picto6")
        .text("100,785 migrants")
        .attr("opacity", "0");

      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".iconPlain").attr("transform", d3.event.transform);
      }

      d3.selectAll(".iconPlain")
        .transition()
        .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity
            .translate(0.4 * this.width, this.height / 20)
            .scale(this.width / 500)
        );

      d3.csv(bar_data, d3.autoType).then((data) => {
        var y = d3
          .scaleBand()
          .range([this.height / 6, this.height])
          .padding(0.1);

        var x = d3.scaleLinear().range([0, 0.15 * this.width]);

        x.domain([
          0,
          d3.max(data, function (d) {
            return d["Dead Missing"];
          }),
        ]);
        y.domain(
          data.map(function (d) {
            return d.Year;
          })
        );

        console.log(
          data.map(function (d) {
            return d.Year;
          })
        );

        let colors = d3
          .scaleOrdinal()
          .domain(data.map((d) => d["Dead Missing"]))
          .range([
            "#acaba8",
            "#a09f9b",
            "#94938f",
            "#5f5f5b",
            "#7b7a75",
            "#898883",
          ]);

        let yScales = [
          this.height - this.height / 1.29 - 30,
          this.height - this.height / 1.35 - 10,
          this.height - this.height / 1.5 - 10,
          this.height - this.height / 1.8 - 10,
          this.height - this.height / 5 - 10,
          this.height - this.height / 9,
        ];
        let bar = this.svg.selectAll("g.bar").data(data).enter().append("g");

        bar
          .append("text")
          .attr("fill", (d) => colors(d["Dead Missing"]))
          .attr("x", 0.8 * this.width)
          .text((d) => d["Dead Missing"])
          .attr("y", function (d, i) {
            console.log("yScales ", yScales[i]);
            return yScales[i];
          })
          .attr("class", (d) => "rect" + d.Year)
          .attr("opacity", 0)
          .style("font-size", "27px")
          .style("font-weight", "bolder")
          .style("font-family", "khula");

        bar
          .append("text")
          .attr("fill", (d) => colors(d["Dead Missing"]))
          .attr("x", 0.8 * this.width)
          .text("Missing or Dead Migrants")
          .attr("y", function (d, i) {
            console.log("yScales ", yScales[i]);
            return yScales[i] + 10;
          })
          .attr("class", (d) => "rect" + d.Year)
          .attr("opacity", 0)
          .style("font-size", "15px")
          .style("font-family", "khula");
      });
    },
    drawInitial: function () {
      this.svg = d3
        .select("#map")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);

      d3.selectAll(".header_navItem.open-modal1").on("click", function () {
        d3.selectAll(".modal1").style("opacity", 1);
        console.log("open the modal");
      });

      const projection = d3
        .geoMercator()
        .fitSize([this.width, this.height], map_json);

      console.log();

      const path = d3.geoPath().projection(projection);
      let map_initial = this.svg
        .append("g")
        .selectAll(".country")
        .data(map_json.features)
        .enter();

      map_initial
        .append("path")
        .attr("d", path)
        .attr("class", "country")
        .attr("fill", "#c4c1b6")
        .attr("stroke", "#898883");

      d3.selectAll(".country")
        .attr("transform", `translate(${this.width / 5},40)`)
        .attr("opacity", 0);

      let tunis = [12, 33];
      let tunis_mid = [12.6, 36.41];
      let libya = [17, 25];
      let libya_mid = [18, 35];
      let italy = [15, 43];
      let mor = [-10.5, 25.5];
      let mor_mid = [-15, 29];
      let spain = [-3, 39];
      let Mauritania = [-13.4357, 18];
      let maur_mid = [-19.13, 19];
      let senegal_mid = [-17.71, 17];
      let senegal = [-13.51, 13];
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

      let route1 = this.svg
        .append("g")
        .selectAll(".route1")
        .attr("class", "route1")
        .data([pointsLine1]);

      route1
        .join("path")
        .attr("class", "route1")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      route1
        .join("text")
        .text("Tunisia")
        .attr("x", projection(tunis)[0])
        .attr("y", projection(tunis)[1])
        .attr("class", "countries route1")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      route1
        .join("text")
        .text("Italy")
        .attr("x", projection(italy)[0])
        .attr("y", projection(italy)[1] + 2)
        .attr("class", "countries route1")
        .style("opacity", "0")
        .style("fill", "#115a73 ");
      route1
        .join("text")
        .text("Libya")
        .attr("x", projection(libya)[0] + 3)
        .attr("y", projection(libya)[1])
        .attr("class", "countries route1")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      let route2 = this.svg.selectAll(".route2").data([pointsLine2]);

      route2
        .join("path")
        .attr("class", "route2")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      let route3 = this.svg.selectAll(".route3").data([pointsLine3]);

      route3
        .join("path")
        .attr("class", "route3")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      route3
        .join("text")
        .text("Morroco")
        .attr("x", projection(mor)[0])
        .attr("y", projection(mor)[1])
        .attr("class", "countries route3")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      route3
        .join("text")
        .text("Spain")
        .attr("x", projection(spain)[0])
        .attr("y", projection(spain)[1])
        .attr("class", "countries route3")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      let route4 = this.svg.selectAll(".route4").data([pointsLine4]);

      route4
        .join("path")
        .attr("class", "route4")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      route3
        .join("text")
        .text("Senegal")
        .attr("x", projection(senegal)[0])
        .attr("y", projection(senegal)[1])
        .attr("class", "countries route4")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      route3
        .join("text")
        .text("Canary")
        .attr("x", projection(canary)[0])
        .attr("y", projection(canary)[1])
        .attr("class", "countries route4")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      route3
        .join("text")
        .text("Mauritania")
        .attr("x", projection(Mauritania)[0])
        .attr("y", projection(Mauritania)[1])
        .attr("class", "countries route4")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      this.svg
        .selectAll(".route5")
        .data([pointsLine5])
        .join("path")
        .attr("class", "route5")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      let route6 = this.svg.selectAll(".route6").data([pointsLine6]);

      route6
        .join("path")
        .attr("class", "route6")
        .attr("d", curve)
        .attr("stroke", "#115a73 ")
        .attr("stroke-width", "0.1rem")
        .attr("fill", "none")
        .attr("opacity", "0")
        .attr("stroke-linecap", "round");

      route3
        .join("text")
        .text("Greece")
        .attr("x", projection(greece)[0])
        .attr("y", projection(greece)[1])
        .attr("class", "countries route6")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      route3
        .join("text")
        .text("Turkey")
        .attr("x", projection(turkey)[0])
        .attr("y", projection(turkey)[1])
        .attr("class", "countries route6")
        .style("opacity", "0")
        .style("fill", "#115a73 ");

      d3.selectAll(".countries").style("color", "#115a73 ");

      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Turkey" || d.properties.name === "Greece"
        )
        .transition()
        .duration(1500)
        .attr("fill", "#c4c1b6");

      console.log(
        "tukey ",
        d3.selectAll(".country").filter((d) => d.properties.name === "Turkey")
      );
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
      d3.selectAll(".country")
        .attr("opacity", 1)
        .transition()
        .attr("fill", "#c4c1b6");
    },
    colorCentral: function () {
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

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomLines1 = d3.zoom().on("zoom", zoomedLines1);
      var zoomLines2 = d3.zoom().on("zoom", zoomedLines2);
      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);
      var zoomLines4 = d3.zoom().on("zoom", zoomedLines4);
      var zoomLines5 = d3.zoom().on("zoom", zoomedLines5);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);
      var zoomText = d3.zoom().on("zoom", zoomedText);
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

      function zoomedText() {
        d3.selectAll(".countries").attr("transform", d3.event.transform);
      }

      d3.selectAll(".country")
        .transition()
        // .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity.scale(1.8).translate(this.width / 100, 0)
        );

      d3.selectAll(".countries")
        .transition()
        // .duration(750)
        .call(
          zoomText.transform,
          d3.zoomIdentity.scale(1.8).translate(this.width / 100, 0)
        );

      d3.selectAll(".route1")
        .transition()
        .duration(750)
        .call(zoomLines1.transform, d3.zoomIdentity.scale(1.8));
      d3.selectAll(".route2")
        .transition()
        .duration(750)
        .call(zoomLines2.transform, d3.zoomIdentity.scale(1.8));
      d3.selectAll(".route3")
        .transition()
        .duration(750)
        .call(zoomLines3.transform, d3.zoomIdentity.scale(1.8));

      d3.selectAll(".route4")
        .transition()
        .duration(750)
        .call(zoomLines4.transform, d3.zoomIdentity.scale(1.8));

      d3.selectAll(".route5")
        .transition()
        .duration(750)
        .call(zoomLines5.transform, d3.zoomIdentity.scale(1.8));

      d3.selectAll(".route6")
        .transition()
        .delay(1000)
        .duration(750)
        .call(zoomLines6.transform, d3.zoomIdentity.scale(1.8));

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
        .delay(750)
        .duration(750)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll("text.countries.route1")
        .style("opacity", 1)
        .transition()
        .delay(750)
        .duration(750);

      d3.selectAll(".route2")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route2 + " " + totalLength_route2)
        .attr("stroke-dashoffset", totalLength_route2)
        .transition()
        .delay(750)
        .duration(750)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);
    },
    colorWestern: function () {
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Tunisia" ||
            d.properties.name === "Libya" ||
            d.properties.name === "Italy"
        )
        .transition()
        .duration(1500)
        .attr("fill-opacity", 0.5);

      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Morocco")
        .transition()
        .duration(1500)
        .attr("fill-opacity", "0.5");
      d3.selectAll(".country")
        .filter((d) => d.properties.name === "Spain")
        .transition()
        .duration(1500)
        .attr("fill-opacity", "0.5");

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

      const totalLength_route3 = d3
        .selectAll(".route3")
        .node()
        .getTotalLength();

      d3.selectAll(".route3")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route3 + " " + totalLength_route3)
        .attr("stroke-dashoffset", totalLength_route3)
        .transition()
        .delay(750)
        .duration(750)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll("text.countries.route3")
        .style("opacity", 1)
        .transition()
        .delay(750)
        .duration(750);

      d3.selectAll("text.countries.route1")
        .style("opacity", 0)
        .transition()
        .delay(750)
        .duration(750);
    },
    colorWestern2: function () {
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Morocco" || d.properties.name === "Spain"
        )
        .transition()
        .duration(1500)
        .attr("fill-opacity", "0.5");
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

      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }

      function zoomedText() {
        d3.selectAll(".countrries").attr("transform", d3.event.transform);
      }

      d3.selectAll(".country")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.scale(1.5));

      var zoomLines4 = d3.zoom().on("zoom", zoomedLines4);
      var zoomLines5 = d3.zoom().on("zoom", zoomedLines5);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);
      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);
      var zoomText = d3.zoom().on("zoom", zoomedText);

      function zoomedLines4() {
        d3.selectAll(".route4").attr("transform", d3.event.transform);
      }

      function zoomedLines5() {
        d3.selectAll(".route5").attr("transform", d3.event.transform);
      }

      function zoomedLines6() {
        d3.selectAll(".route6").attr("transform", d3.event.transform);
      }

      function zoomedLines3() {
        d3.selectAll(".route3").attr("transform", d3.event.transform);
      }

      d3.selectAll(".countries")
        .transition()
        // .duration(750)
        .call(
          zoomText.transform,
          d3.zoomIdentity.scale(1.8).translate(this.width / 100, 0)
        );

      d3.selectAll(".route4")
        .transition()
        .duration(750)
        .call(zoomLines4.transform, d3.zoomIdentity.scale(1.5));

      d3.selectAll(".route5")
        .transition()
        .duration(750)
        .call(zoomLines5.transform, d3.zoomIdentity.scale(1.5));

      d3.selectAll(".route6")
        .transition()

        .duration(750)
        .call(zoomLines6.transform, d3.zoomIdentity.scale(1.5));

      d3.selectAll(".route3")
        .transition()
        .duration(750)
        .call(zoomLines3.transform, d3.zoomIdentity.scale(1.5));

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

      d3.selectAll(".countries.route4").style("opacity", "1");

      d3.selectAll("text.countries.route3")
        .style("opacity", 0)
        .transition()
        .delay(750)
        .duration(750);

      d3.selectAll(".route5")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route5 + " " + totalLength_route5)
        .attr("stroke-dashoffset", totalLength_route5)
        .transition()
        .delay(750)
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);

      function zoomedLines3() {
        d3.selectAll(".route3").attr("transform", d3.event.transform);
      }
    },
    colorEastern: function () {
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Morocco" || d.properties.name === "Spain"
        )
        .transition()
        .duration(1500)
        .attr("fill", "#c4c1b6");
      d3.selectAll(".country")
        .filter(
          (d) =>
            d.properties.name === "Mauritania" ||
            d.properties.name === "Senegal"
        )
        .transition()
        .duration(1500)
        .attr("fill-opacity", "0.5");
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

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }
      function zoomedLines6() {
        d3.selectAll(".route6").attr("transform", d3.event.transform);
      }

      function zoomedText() {
        d3.selectAll(".countries").attr("transform", d3.event.transform);
      }

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomLines6 = d3.zoom().on("zoom", zoomedLines6);
      var zoomText = d3.zoom().on("zoom", zoomedText);

      d3.selectAll(".country")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.scale(1.3));

      d3.selectAll(".countries")
        .transition()
        .duration(750)
        .call(zoomText.transform, d3.zoomIdentity.scale(1.3));

      console.log("route 6 is ", d3.selectAll(".route6"));
      d3.selectAll(".route6")
        .transition()
        .duration(750)
        .call(zoomLines6.transform, d3.zoomIdentity.scale(1.3));

      const totalLength_route6 = d3
        .selectAll(".route6")
        .node()
        .getTotalLength();

      d3.selectAll(".route6")
        .attr("opacity", 1)
        .attr("stroke-dasharray", totalLength_route6 + " " + totalLength_route6)
        .attr("stroke-dashoffset", totalLength_route6)
        .transition()
        .delay(750)
        .duration(1000)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll(".countries.route6").style("opacity", "1");

      d3.selectAll(".countries.route4").style("opacity", "0");
    },

    hideChart: function () {
      d3.selectAll(".second").attr("opacity", "0").transition();
      d3.selectAll(".text-years").style("opacity", "0").transition();
    },

    highlight2014: function () {
      d3.selectAll(".year1").attr("fill", "#766F81").transition();
      d3.selectAll(".picto1").transition().attr("opacity", 1);
      d3.selectAll(".year").style("opacity", "1").html("2014");

      var div1 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("opacity", 0);
      var div2 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);

      var div3 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip3")
        .style("opacity", 0);
      var div4 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip4")
        .style("opacity", 0);

      var div5 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);
      var div6 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);
      var div7 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);
      var div8 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);
      var div9 = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip2")
        .style("opacity", 0);

      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if ((d < 382 && d > 380) || (d < 378 && d > 376)) {
            return "iconSelectable_2014";
          } else if (d < 401 && d > 399) {
            console.log(d);
            return "legend_year";
          } else if (d < 402 && d > 400) {
            return "legend_select";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else return "iconPlain";
        });

      d3.selectAll(".rect2014").attr("opacity", 1);
      d3.selectAll(".text2014").attr("opacity", 1);

      console.log("uses ", d3.selectAll("use"));

      let evros =
        "https://s3.eu-central-1.amazonaws.com/euobs-media/b9f57a411d15045489f54ba555cdae87.jpg";
      let refugee =
        "https://www.middleeasteye.net/sites/default/files/styles/article_page/public/images-story/greece_turkey-afp-2_29_20.jpeg?itok=IYj15sgC";
      let syrian_ref =
        "https://s.abcnews.com/images/International/rt_refugee_dinghy_deflates_05_jc_150914_16x9_992.jpg";
      let yarmouk =
        "https://assets.irinnews.org/s3fs-public/styles/responsive_large/public/images/201502021138530732.jpg?lyVxDDjNVQhgb_iIAtQmlc2Z_BJA_I9y&itok=UHdwIk85";

      d3.selectAll("use")
        .on("mouseover", function (d, i) {
          if (d < 382 && d > 380) {
            div1.style("opacity", 1);
            div1.html(
              "<img src=" +
                evros +
                " alt='boat'/> <br> the Barbed Wire Evros Fence."
            );
            div1
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          } else if (d < 378 && d > 376) {
            div2.style("opacity", 0.9);
            div2.html(
              "<img src=" +
                refugee +
                " alt='boat'/> <br> It is estimated that most people coming into Greece from Turkey qualify for refugee status in most European countries."
            );
            div2
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          }
          if (d < 342 && d > 340) {
            div3.style("opacity", 1);
            div3.html(
              "<img src=" +
                syrian_ref +
                " alt='boat'/> <br> The number of Syrian refugees reached 4 millions in 2015. These refugees were living in overcrowded and underfunded camps in countries like Turkey and Lebanon"
            );
            div3
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          } else if (d > 300 && d < 302) {
            div4.style("opacity", 0.9);
            div4.html(
              "<img src=" +
                yarmouk +
                " alt='boat'/> <br> Tariq, a 19-year-old Palestinian from Yarmouk Camp, close to the Syrian capital, Damascus, whom Human Rights Watch interviewed in Serbia, explained, “I left so I could escape from the war and out of fear of being arrested, just like happened to my family members. Also, I was afraid of the army. The situation of Palestinians in Syria is particularly difficult…Yarmouk camp is under siege. There is no food or anything.” https://www.hrw.org/report/2015/06/19/mediterranean-migration-crisis/why-people-flee-what-eu-should-do\
                Above image is from the Yarmouk Camp in Syria"
            );
            div4
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          } else if (d > 276 && d < 278) {
            div5.style("opacity", 0.9);
            div5.html(
              "<img src=" +
                refugee +
                " alt='boat'/> <br> It is estimated that most people coming into Greece from Turkey qualify for refugee status in most European countries."
            );
            div5
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          } else if (d > 207 && d < 209) {
            div6.style("opacity", 0.9);
            div6.html(
              "<img src=" +
                refugee +
                " alt='boat'/> <br> It is estimated that most people coming into Greece from Turkey qualify for refugee status in most European countries."
            );
            div6
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          } else if (d > 254 && d < 256) {
            div7.style("opacity", 0.9);
            div7.html(
              "<img src=" +
                refugee +
                " alt='boat'/> <br> It is estimated that most people coming into Greece from Turkey qualify for refugee status in most European countries."
            );
            div7
              .style("left", d3.event.pageX + "px")
              .style("top", d3.event.pageY - 150 + "px");
          }
        })

        .on("mouseout", function (d) {
          div1.transition().duration(500).style("opacity", 0);
          div2.transition().duration(500).style("opacity", 0);
          div3.transition().duration(500).style("opacity", 0);
          div4.transition().duration(500).style("opacity", 0);
          div5.transition().duration(500).style("opacity", 0);
          div6.transition().duration(500).style("opacity", 0);
          div7.transition().duration(500).style("opacity", 0);
        });

      d3.selectAll(".picto2").transition().attr("opacity", 0);
    },

    highlight2015: function () {
      d3.selectAll(".year").html("2015");
      d3.selectAll(".year2").attr("fill", "#766F81").transition();
      d3.selectAll(".first_text").attr("opacity", "0");
      d3.selectAll(".evros").transition().delay(500).attr("opacity", "0");
      d3.selectAll(".first_text").transition().delay(500).attr("opacity", "0");

      d3.selectAll(".picto2").transition().attr("opacity", 1);

      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if ((d < 382 && d > 380) || (d < 378 && d > 376)) {
            return "iconSelectable_2014";
          } else if (
            (d > 340 && d < 342) ||
            (d > 300 && d < 302) ||
            (d > 276 && d < 278) ||
            (d > 207 && d < 209) ||
            (d > 254 && d < 256)
          ) {
            return "iconSelectable_2015";
          } else if (
            d < 400 - 42.14548797 &&
            d > 400 - 42.14548797 - 204.4946677
          ) {
            return "iconSelected2015";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else return "iconPlain";
        });

      d3.selectAll(".picto3").transition().attr("opacity", 0);

      d3.selectAll(".rect2015").attr("opacity", 1);

      d3.selectAll(".text2015").attr("opacity", 1);
    },

    highlight2016: function () {
      d3.selectAll(".year").html("2016");
      d3.selectAll(".picto3").transition().attr("opacity", 1);
      d3.selectAll(".year3").attr("fill", "#766F81").transition();
      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if ((d < 382 && d > 380) || (d < 378 && d > 376)) {
            return "iconSelectable_2014";
          } else if (
            (d > 340 && d < 342) ||
            (d > 300 && d < 302) ||
            (d > 276 && d < 278) ||
            (d > 207 && d < 209) ||
            (d > 254 && d < 256)
          ) {
            return "iconSelectable_2015";
          } else if (d > 143 && d < 145) {
            return "iconSelectable_2016";
          } else if (
            d < 400 - 42.14548797 &&
            d > 400 - 42.14548797 - 204.4946677
          ) {
            return "iconSelected2015";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101
          ) {
            return "iconSelected2016";
          } else return "iconPlain";
        });
      d3.selectAll(".picto4").transition().attr("opacity", 0);
      d3.selectAll(".rect2016").attr("opacity", 1);
      d3.selectAll(".text2016").attr("opacity", 1);
    },

    highlight2017: function () {
      d3.selectAll(".year").html("2017");
      d3.selectAll(".year4").attr("fill", "#766F81").transition();
      d3.selectAll(".year3").attr("fill", "#766F81").transition();
      d3.selectAll(".picto4").transition().attr("opacity", 1);
      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if ((d < 382 && d > 380) || (d < 378 && d > 376)) {
            return "iconSelectable_2014";
          } else if (
            (d > 340 && d < 342) ||
            (d > 300 && d < 302) ||
            (d > 276 && d < 278) ||
            (d > 207 && d < 209) ||
            (d > 254 && d < 256)
          ) {
            return "iconSelectable_2015";
          } else if (d > 143 && d < 145) {
            return "iconSelectable_2016";
          } else if (d > 100 && d < 98) {
            return "iconSelectable_2017";
          } else if (
            d < 400 - 42.14548797 &&
            d > 400 - 42.14548797 - 204.4946677
          ) {
            return "iconSelected2015";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101
          ) {
            return "iconSelected2016";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 - 71.66341101 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101 - 34.38417303
          ) {
            return "iconSelected2017";
          } else return "iconPlain";
        });
      d3.selectAll(".picto5").transition().attr("opacity", 0);
      d3.selectAll(".rect2017").attr("opacity", 1);

      d3.selectAll(".text2017").attr("opacity", 1);
    },

    highlight2018: function () {
      d3.selectAll(".year").html("2018");
      d3.selectAll(".year5").attr("fill", "#766F81").transition();
      d3.selectAll(".picto5").transition().attr("opacity", 1);
      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if ((d < 382 && d > 380) || (d < 378 && d > 376)) {
            return "iconSelectable_2014";
          } else if (
            (d > 340 && d < 342) ||
            (d > 300 && d < 302) ||
            (d > 276 && d < 278) ||
            (d > 207 && d < 209) ||
            (d > 254 && d < 256)
          ) {
            return "iconSelectable_2015";
          } else if (d > 143 && d < 145) {
            return "iconSelectable_2016";
          } else if (d > 100 && d < 98) {
            return "iconSelectable_2017";
            // } else if (d > 79 && d < 81) {
            //   return "iconSelectable_2018";
            // }
          } else if (
            d < 400 - 42.14548797 &&
            d > 400 - 42.14548797 - 204.4946677
          ) {
            return "iconSelected2015";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101
          ) {
            return "iconSelected2016";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 - 71.66341101 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101 - 34.38417303
          ) {
            return "iconSelected2017";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 - 71.66341101 - 34.38417303 &&
            d >
              400 -
                42.14548797 -
                204.4946677 -
                71.66341101 -
                34.38417303 -
                23.07187056
          ) {
            return "iconSelected2018";
          } else return "iconPlain";
        });
      d3.selectAll(".picto6").transition().attr("opacity", 0);
      d3.selectAll(".rect2018").attr("opacity", 1);
      d3.selectAll(".text2018").attr("opacity", 1);
    },
    highlight2019: function () {
      d3.selectAll(".year").html("2019");
      d3.selectAll(".year6").attr("fill", "#766F81").transition();
      d3.selectAll(".picto6").transition().attr("opacity", 1);
      d3.selectAll("use")
        .transition()
        .attr("class", function (d, i) {
          if (d < 400 - 42.14548797 && d > 400 - 42.14548797 - 204.4946677) {
            return "iconSelected2015";
          } else if (d < 400 && d > 400 - 42.14548797) {
            return "iconSelected2014";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101
          ) {
            return "iconSelected2016";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 - 71.66341101 &&
            d > 400 - 42.14548797 - 204.4946677 - 71.66341101 - 34.38417303
          ) {
            return "iconSelected2017";
          } else if (
            d < 400 - 42.14548797 - 204.4946677 - 71.66341101 - 34.38417303 &&
            d >
              400 -
                42.14548797 -
                204.4946677 -
                71.66341101 -
                34.38417303 -
                23.07187056
          ) {
            return "iconSelected2018";
          } else if (
            d <
            400 -
              42.14548797 -
              204.4946677 -
              71.66341101 -
              34.38417303 -
              23.07187056
          ) {
            return "iconSelected2019";
          }
          // }
        });
      d3.selectAll(".legend_year").attr("class", "iconPlain");
      d3.selectAll(".picto6").transition().attr("opacity", 1);
      d3.selectAll(".rect2019").attr("opacity", 1);
      d3.selectAll(".text2019").attr("opacity", 1);
    },
    lastView: function () {
      d3.selectAll(".last").transition().attr("opacity", "1");
      d3.selectAll(".year").transition().style("opacity", "0");
      d3.selectAll(".observ").transition().style("opacity", "1");
    },

    removeHighlight: function () {
      d3.selectAll(".iconPlain").transition().attr("opacity", "0");
      d3.selectAll(".icon-text1").transition().attr("opacity", "0");
      d3.selectAll(".icon-text2").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2014").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2015").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2016").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2017").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2018").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2019").transition().attr("opacity", "0");
      d3.selectAll(".iconSelected2020").transition().attr("opacity", "0");
      d3.selectAll(".iconSelectable").transition().attr("opacity", "0");
      d3.selectAll(".picto6").transition().attr("opacity", 0);
      d3.selectAll(".picto5").transition().attr("opacity", 0);
      d3.selectAll(".picto4").transition().attr("opacity", 0);
      d3.selectAll(".picto3").transition().attr("opacity", 0);
      d3.selectAll(".picto2").transition().attr("opacity", 0);
      d3.selectAll(".picto1").transition().attr("opacity", 0);
      d3.selectAll(".year").transition().style("opacity", "0");
      d3.selectAll(".observ").transition().style("opacity", 1);
      d3.selectAll("rect").transition().style("opacity", 0);
      d3.selectAll(".text2014").transition().style("opacity", 0);
      d3.selectAll(".text2015").transition().style("opacity", 0);
      d3.selectAll(".text2016").transition().style("opacity", 0);
      d3.selectAll(".text2017").transition().style("opacity", 0);
      d3.selectAll(".text2018").transition().style("opacity", 0);
      d3.selectAll(".text2019").transition().style("opacity", 0);
      d3.selectAll(".countries").transition().style("opacity", 0);
    },

    showIcons: function () {
      d3.selectAll(".footer").transition().duration(1000).style("opacity", "1");
      d3.selectAll(".iconPlain")
        .transition()
        .duration(1000)
        .attr("opacity", "1");
      d3.selectAll(".icon-text1")
        .transition()
        .duration(1000)
        .attr("opacity", "2");
      d3.selectAll(".icon-text2")
        .transition()
        .duration(1000)
        .attr("opacity", "1");

      d3.selectAll(".picto1").transition().attr("opacity", 0);
      d3.selectAll(".legend_year").attr("opacity", 1);
      d3.selectAll(".legend").style("opacity", 1);
      d3.selectAll(".legend_select").attr("opacity", 1);
    },
    hideRoute12: function () {
      d3.selectAll(".route1").transition().duration(200).attr("opacity", 0);
      d3.selectAll(".route2").transition().duration(200).attr("opacity", 0);
    },
    hideRoute3: function () {
      d3.selectAll(".route3").transition().attr("opacity", "0");
    },
    hideRoute45: function () {
      d3.selectAll(".route4").transition().duration(200).attr("opacity", 0);
      d3.selectAll(".route5").transition().duration(200).attr("opacity", 0);
    },
    hideRoute6: function () {
      d3.selectAll(".route6").transition().duration(200).attr("opacity", 0);
      d3.selectAll(".countries.route6")
        .transition()
        .duration(200)
        .style("opacity", 0);
    },
  },
};
</script>

<style>
#sections {
  position: relative;
  /* display: inline-block; */
  width: 30%;
  height: 40%;
  /* top: 60px; */
  z-index: 90;
  margin-left: 1rem;
  /* height: 10000px; */
  padding: 1%;
}

@media (max-width: 900px) {
  #sections {
  }
}

.step {
  margin-bottom: 1rem;
  height: 40%;
  /* font-family: "Domine"; */
  font-family: "Biryani";
  font-weight: 300;
  line-height: 1.4em;
  font-size: 1.5em;
  text-align: justify;
  margin-left: 0.5rem;
  display: flex;
  top: 20%;
  flex-direction: column;
  justify-content: space-around;
  background-color: none;
  padding-left: 2%;
}

.countries {
  font-family: "khula";
  line-height: 1.4em;
  font-size: 0.6em;
  font-weight: bold;
  opacity: 0.6;
}

h2 {
  font-family: "Khula";
  font-size: 2em;
  opacity: 1;
  text-align: left;
  fill: #444444;
}

h3 {
  font-family: "Biryani";
  line-height: 1.4em;
  font-size: 1.1em;
  padding: auto;
  opacity: 1;
  text-align: left;
  fill: #444444;
}

@media (max-width: 900px) {
  .step {
  }
}
#graphic,
#graphic2,
#graphic3 {
  width: 100%;
  flex-direction: row;
  align-items: top;
  justify-content: space-around;
  padding-right: 2px;
}

@media (max-width: 900px) {
  #graphic {
    width: 100rem;
    flex-direction: row;
    align-items: top;
    justify-content: space-around;
  }
}

#map,
#bub,
#bub2 {
  position: fixed;
  top: 1rem;
  z-index: 1;
  padding: 2px;
}

img {
  width: 200px;
  height: 100px;
  display: block;
  margin-left: auto;
  margin-right: auto;
  border-radius: 8px;
}

.tooltip1 {
  min-width: 200px;
  max-width: 400px;
  height: 100px;
  top: 50%;
  left: 50%;
  margin-left: 20px;
  transform: translate(0, 50%);
  padding: 0;
  color: #eeeeee;
  background-color: #444444;
  font-weight: normal;
  font-size: 13px;
  border-radius: 8px;
  position: absolute;
  z-index: 99999999;
  box-sizing: border-box;
  box-shadow: 0 1px 8px rgba(0, 0, 0, 0.5);
  visibility: hidden;
  opacity: 1;
  transition: opacity 0.8s;
}

.tooltip1 img {
}
.text-content {
  padding: 10px 20px;
}

.tooltip1 i {
  position: absolute;
  top: 50%;
  right: 100%;
  margin-top: -12px;
  width: 12px;
  height: 24px;
  overflow: hidden;
}
.tooltip1 i::after {
  content: "";
  position: absolute;
  width: 12px;
  height: 12px;
  left: 0;
  top: 50%;
  transform: translate(50%, -50%) rotate(-45deg);
  background-color: #444444;
  box-shadow: 0 1px 8px rgba(0, 0, 0, 0.5);
}

#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #dbd9d2;
  margin-top: 1rem;
  color: #898883;
  overflow: hidden;
}

.scroll-container {
  height: 150vh;
  scroll-snap-type: y mandatory;
}

section {
  height: 100vh;
  scroll-snap-align: center;
}

header {
  align-items: center;
  background: #898883;
  color: #dbd9d2;
  display: flex;
  height: 60px;
  justify-content: space-between;
  padding: 0 20px;
  position: fixed;
  width: 95%;
  z-index: 100;
  margin-top: 0;
  top: 4px
}

.header_title {
  font-size: 17px;
  font-weight: 500;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  font-family: "Raleway";
}

.header_nav {
  list-style: none;
  padding: 0;
  text-decoration: none;
}

.header_navItem {
  display: inline-block;
  font-size: 12px;
  letter-spacing: 0.1em;
  margin-left: 20px;
  text-transform: uppercase;
  color: #dbd9d2;
  text-decoration: none;
  font-family: "Khula";
  background-color: #898883;
  border: none;
}

.header_navItem a {
  text-decoration: none;
  color: #dbd9d2;
}

.header_navItem a:after {
  display: block;
  height: 1px;
  margin-bottom: 2px;
  margin-top: 2px;
  transition: height 100ms ease-in, margin 100ms ease-in;
  width: 100%;
}

.header_navItem a:hover:after {
  height: 3px;
  margin-bottom: 0;
  color: #dbd9d2;
}

.icon-text1 {
  fill: #898883;
  text-anchor: right;
  text-align: justify;
  font-size: 1.3em;
  font-family: "khula";
  font-weight: bold;
  border-style: 2px solid black;
}

.icon-text2 {
  fill: #898883;
  text-anchor: right;
  text-align: justify;
  font-size: 0.9em;
  font-family: "khula";
  font-weight: bold;
  border-style: 2px solid black;
}

.iconPlain {
  fill: #a7a59b;
}

.iconSelected2019 {
  fill: #7bccc4;
}

.picto6 {
  fill: #7bccc4;
  font-size: 1em;
  font-weight: bold;
}

.legend {
  fill: #898883;
  font-size: 1em;
  font-weight: bold;
  font-family: "khula";
}

.iconSelected2018 {
  fill: #4ab9ae;
}

.picto5 {
  fill: #4ab9ae;
  font-size: 1em;
  font-weight: bold;
}

.iconSelected2017 {
  fill: #55a190;
  stroke-width: 10px;
}

.picto4 {
  fill: #55a190;
  font-size: 1em;
  font-weight: bold;
}

.iconSelected2016 {
  fill: #4eb3d3;
}

.picto3 {
  fill: #4eb3d3;
  font-size: 1em;
  font-weight: bold;
}

.iconSelected2015 {
  fill: #2b8cbe;
}

.picto2 {
  fill: #2b8cbe;
  font-size: 1em;
  font-weight: bold;
}

.iconSelected2014 {
  fill: #08589e;
}

.picto1 {
  fill: #08589e;
  font-size: 1em;
  font-weight: bold;
}

.iconSelectedMissing {
  fill: #fff1e0;
}

.iconSelectable_2019 {
  /* opacity: 0.5; */
  fill: #7bccc4;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}
.iconSelectable_2018 {
  /* opacity: 0.5; */
  fill: #4ab9ae;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}
.iconSelectable_2017 {
  /* opacity: 0.5; */
  fill: #55a190;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}
.iconSelectable_2016 {
  /* opacity: 0.5; */
  fill: #4eb3d3;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}
.iconSelectable_2015 {
  /* opacity: 0.5; */
  fill: #2b8cbe;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}
.iconSelectable_2014 {
  /* opacity: 0.5; */
  fill: #08589e;
  stroke-width: 0.6;
  stroke: black;
  opacity: 0.8;
}

.legend_year {
  fill: #898883;
}

.legend_select {
  stroke-width: 0.6;
  stroke: black;
  fill: #a7a59b;
}

.intro,
.banksy {
  position: relative;
  justify-content: center;
  align-items: center;
  top: 15%;
  left: 0.5%;
  padding-left: 5%;
}

#progress-container-el {
  background-color: #dbd9d2 !important;
  height: 20px;
}

#progress-el {
  background-color: #766f81 !important;
  height: 20px;
}

.landing {
  display: flex;
  top: -4px
}

.banksy {
  flex: 0.7;
}
.intro {
  flex: 1;
}

.landing {
  background-image: url("../public/Data/inspo.jpg");

  background-position: left;

  background-repeat: no-repeat;

  background-attachment: fixed;

  background-size: 100;

  background-color: #dbd9d2;
}

.content {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-15%, -20%);
  color: #898883;
}

div.tooltip {
  position: absolute;
  text-align: center;
  width: 200px;
  height: 130px;
  padding: 2px;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
  z-index: 50;
}

div.tooltip2 {
  position: absolute;
  text-align: center;
  width: 200px;
  height: 200px;
  padding: 2px;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
  z-index: 50;
}

div.tooltip3 {
  position: absolute;
  text-align: center;
  width: 200px;
  height: 200px;
  padding: 2px;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
  z-index: 50;
}

div.tooltip4 {
  position: absolute;
  text-align: center;
  width: 300px;
  height: 400px;
  padding: 2px;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
  z-index: 50;
}

div.year {
  position: absolute;
  font-size: 40px;
  top: 15%;
  left: 3.5%;
  opacity: 0;
  color: #898883;
  text-align: left;
  font-family: "biryani";
}

div.observ {
  font-size: 30px;
  color: #898883;
}

.countries {
  fill: #766f81;
}

.footer {
  align-items: center;
  background: #898883;
  color: #dbd9d2;
  display: flex;
  height: 30px;
  justify-content: space-between;
  padding: 0 20px;
  top: 67px;
  position: fixed;
  transition: transform 300ms ease-in-out;
  transform: translateY(0);
  width: 95%;
  z-index: 100;
  text-decoration: none;
  font-family: "Courier New", Courier;
  opacity: 0;
}

.f1 {
  background: #898883;
  padding: 0;
  margin: 0;
}

.f2 {
  background: #898883;
}

.f3 {
  background: #898883;
}

.f1,
.f2,
.f3 {
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  display: inline-block;
  text-align: left;
  padding: 0 2%;
  min-width: 300px;
  height: 25px;
  font-size: 15px;
  font-weight: bold;
}

.f2 {
  border-right: 2px solid #dbd9d2;
  width: 70%;
}

.f1 {
  text-align: left;
  border-right: 2px solid #dbd9d2;
  width: 45%;
}


</style>
