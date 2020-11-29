<template>
  <main class="scroll-container">
    <!-- <section>
        <h2></h2>
    </section> -->
    <header>
      <h1 class="header_title">
        Mapping State Violence and Colonial Legacies in the Mediterranean
      </h1>
      <nav>
        <ul class="header_nav">
          <li class="header_navItem">
            <a href="#">About</a>
          </li>
          <li class="header_navItem">
            <a href="#">Contact</a>
          </li>
        </ul>
      </nav>
    </header>
    <div id="app">
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
        <div id="bub"></div>
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

export default {
  name: "App",
  components: {
    MapIntro,
    Story,
    Story2,
    Story3,
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
      svg2: d3
        .select("#bub")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height),
    };
  },
  mounted() {
    console.log(bar_data);
    this.drawInitial();
    this.drawInitialBub();
    // this.drawSecondBub();

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

      console.log("scroll func ", scroll_functions);

      let sign = activeIndex - lastIndex < 0 ? -1 : 1;
      let scrolledSections = d3.range(
        lastIndex + sign,
        activeIndex + sign,
        sign
      );

      console.log("scrolled sec ", scrolledSections);
      scrolledSections.forEach((i) => {
        console.log("function ", i);
        if (i == 0) {
          scroll_functions.hideMap();
        }
        if (i == 1) {
          scroll_functions.drawMap();
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
          scroll_functions.hideChart();
          scroll_functions.removeHighlight();
        } else if (i == 6) {
          scroll_functions.hideMap();
          scroll_functions.hideRoute6();
          // scroll_functions.firstImage();

          // scroll_functions.drawBarChart();
          //  scroll_functions.zoomBackChart()

          scroll_functions.showIcons();
        } else if (i == 7) {
          // scroll_functions.unzoomChart();
          scroll_functions.highlight2014();
        } else if (i == 8) {
          scroll_functions.highlight2015();
        } else if (i == 9) {
          scroll_functions.highlight2016();
        } else if (i == 10) {
          scroll_functions.highlight2017();
        } else if (i == 11) {
          scroll_functions.highlight2018();
        } else if (i == 12) {
          scroll_functions.highlight2019();
        } else if (i == 13) {
          scroll_functions.drawSecondBub();
          scroll_functions.removeHighlight();
          scroll_functions.lastView()
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
    drawSecondBub: function () {
      this.svg = d3
        .select("#bub2")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);

      d3.csv(missing_data, d3.autoType).then((data) => {
        let color = d3.scaleSequential(
          d3.extent(data, (d) => d["Reported Year"]),
          d3.interpolateCividis
        );
        let yAxis = (g) =>
          g
            .call(d3.axisLeft(y).ticks(8))
            .call((g) => g.select(".domain").remove())
            .call((g) => g.selectAll(".tick line").remove());

        let xAxis = (g) =>
          g
            .call(d3.axisTop(x))
            .call((g) => g.select(".domain").remove())
            .call((g) =>
              g
                .append("text")
                .attr("x", this.width)
                .attr("y", 20)
                .attr("font-weight", "bold")
                .attr("fill", "currentColor")
                .attr("text-anchor", "end")
                .text("Month")
            );

        let r = d3
          .scaleSqrt()
          .domain(d3.extent(data, (d) => d["Total Dead and Missing"]))
          .range([1, 10]);

        let y = d3
          .scaleLinear()
          .domain(d3.extent(data, (d) => d["Reported Year"]))
          .range([this.height / 8, this.height / 1.5]);

        data.forEach(function (d) {
          if (d["Reported Month"] == "Jan") {
            d.month = 1;
          }
          if (d["Reported Month"] == "Feb") {
            d.month = 2;
          }
          if (d["Reported Month"] == "Mar") {
            d.month = 3;
          }
          if (d["Reported Month"] == "Apr") {
            d.month = 4;
          }
          if (d["Reported Month"] == "May") {
            d.month = 5;
          }
          if (d["Reported Month"] == "Jun") {
            d.month = 6;
          }
          if (d["Reported Month"] == "Jul") {
            d.month = 7;
          }
          if (d["Reported Month"] == "Aug") {
            d.month = 8;
          }
          if (d["Reported Month"] == "Sep") {
            d.month = 9;
          }
          if (d["Reported Month"] == "Oct") {
            d.month = 10;
          }
          if (d["Reported Month"] == "Nov") {
            d.month = 11;
          }
          if (d["Reported Month"] == "Dec") {
            d.month = 12;
          }
        });

        let x = d3
          .scaleLinear()
          .domain(d3.extent(data, (d) => d.month))
          .range([this.width / 8, this.width / 1.5]);

        console.log("x is ", x);

        // let groups = d3.group(data, (d) => d.year);


        let noSplitHeight = 500;
        let splitHight = 900;

        const wrapper = this.svg.append("g");
        // .attr("transform", `translate(${margin.left}, ${margin.top})`);
        console.log("wrapper is ", wrapper);
        // Add x-Axis
        wrapper.append("g").call(xAxis);

        const yAxisContainer = wrapper
          .append("g")
          .attr("transform", `translate(-10,0)`);

        const circles = wrapper
          .append("g")
          .attr("class", "last")
          .selectAll("circle")
          .data(data)
          .join("circle")
          .attr("r", (d) => r(d["Total Dead and Missing"]))
          .attr("fill", (d) => color(d["Reported Year"]))
          .attr("cy", (d) => y(d["Reported Year"]))
          .attr("cx", (d) => x(d.month))
          .attr("opacity", "0");

        let force = d3
          .forceSimulation(data)
          .force("charge", d3.forceManyBody().strength(0))
          .force(
            "y",
            d3.forceY((d) => y(d["Reported Year"]))
          )
          .force(
            "x",
            d3.forceX((d) => x(d.month))
          )
          .force(
            "collision",
            d3.forceCollide().radius((d) => r(d["Total Dead and Missing"]) + 1)
          );

        // force.on("tick", () => {
        //   circles
        //     .transition()
        //     .ease(d3.easeLinear)
        //     .attr("cx", (d) => d.x)
        //     .attr("cy", (d) => d.y);
        // });

        //  invalidation.then(() => force.stop());

        // return Object.assign(svg.node(), {
        //   update(split) {
        //     let height = split ? splitHeight : noSplitHeight;
        //     let years = [...yearGroups.keys()].sort()

        //     // Update height of svg object
        //     const t = d3.transition().duration(750);
        //     svg.transition(t).attr('viewBox', [0, 0, width, height]);

        //     // Update domain of y-Axis
        //     y.domain(split ? years : ['All']);
        //     y.range(split ? [splitHeight , 0] : [noSplitHeight , 0]);
        //     yAxisContainer.call(yAxis, y, split ? years : ['All'])
        //       .call(g => g.select('.domain').remove())
        //       .call(g => g.selectAll('.tick line').remove());

        //     // Update simulation
        //     force.force('y', split ? d3.forceY(d => y(d.year) + y.bandwidth() / 2) : // If split by year align by year
        //                              d3.forceY((noSplitHeight) / 2)); // If not split align to middle
        //     //force.nodes(running);
        //     force.alpha(1).restart();
        //   }
        // })
      });
    },

    drawInitialBub: function () {
      this.svg = d3
        .select("#bub")
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);

      d3.csv(bar_data, d3.autoType).then((data) => {
        this.width = window.innerWidth;
        this.height = window.innerHeight;
        var x = d3
          .scaleLinear()
          .domain([2014, 2019])
          .range([this.width / 6, this.width / 1.5]);

        let g1 = this.svg.append("g");
        let g2 = this.svg.append("g");
        let g3 = this.svg.append("g");
        let g4 = this.svg.append("g");
        let g5 = this.svg.append("g");
        let g6 = this.svg.append("g");
        var y = d3
          .scaleLinear()
          .domain([2014, 2020])
          .range([this.height / 1.5, this.height / 7]);

        var z = d3
          .scaleSqrt()
          .domain([0, d3.max(data, (d) => d["Total Arrivals"])])
          .range([20, 100]);

        const [width, height, margin] = [this.width, 250, 15];

        const radius = z(data[0]["Total Arrivals"]);
        console.log(radius);
        const xStart = x(data[0]["Year"] + radius / 100);
        const yStart = height * 1.5 - radius;
        const circle = g1
          .append("path")
          .attr("class", "second year1")
          .attr(
            "d",
            `M ${xStart},${yStart}
                a ${radius} ${radius} 0 1 1 0 ${radius * 2}
                  ${radius} ${radius} 0 1 1 0 ${-radius * 2} z`
          );
        g1.append("text")
          .text("2014")
          .attr("x", xStart)
          .attr("y", yStart - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");

        const radius2 = z(data[1]["Total Arrivals"]);
        const xStart2 = x(data[1]["Year"] + radius2 / 100 - 0.4);
        const yStart2 = height * 1.5 - radius2;
        const circle2 = g2
          .append("path")
          .attr("class", "second year2")
          .attr(
            "d",
            `M ${xStart2},${yStart2}
                a ${radius2} ${radius2} 0 1 1 0 ${radius2 * 2}
                  ${radius2} ${radius2} 0 1 1 0 ${-radius2 * 2} z`
          );
        g2.append("text")
          .text("2015")
          .attr("x", xStart2)
          .attr("y", yStart2 - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");

        const radius3 = z(data[2]["Total Arrivals"]);
        const xStart3 = x(data[2]["Year"] + radius3 / 100);
        const yStart3 = height * 1.5 - radius3;
        const circle3 = g3
          .append("path")
          .attr("class", "second year3")
          .attr(
            "d",
            `M ${xStart3},${yStart3}
                a ${radius3} ${radius3} 0 1 1 0 ${radius3 * 2}
                  ${radius3} ${radius3} 0 1 1 0 ${-radius3 * 2} z`
          );
        g3.append("text")
          .text("2016")
          .attr("x", xStart3)
          .attr("y", yStart3 - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");
        const radius4 = z(data[3]["Total Arrivals"]);
        const xStart4 = x(data[3]["Year"] + radius4 / 100);
        const yStart4 = height * 1.5 - radius4;
        const circle4 = g4
          .append("path")
          .attr("class", "second year4")
          .attr(
            "d",
            `M ${xStart4},${yStart4}
                a ${radius4} ${radius4} 0 1 1 0 ${radius4 * 2}
                  ${radius4} ${radius4} 0 1 1 0 ${-radius4 * 2} z`
          );

        g4.append("text")
          .text("2017")
          .attr("x", xStart4)
          .attr("y", yStart4 - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");

        const radius5 = z(data[4]["Total Arrivals"]);
        const xStart5 = x(data[4]["Year"] + radius5 / 100 - 0.2);
        const yStart5 = height * 1.5 - radius5;
        const circle5 = g5
          .append("path")
          .attr("class", "second year5")
          .attr(
            "d",
            `M ${xStart5},${yStart5}
                a ${radius5} ${radius5} 0 1 1 0 ${radius5 * 2}
                  ${radius5} ${radius5} 0 1 1 0 ${-radius5 * 2} z`
          );

        g5.append("text")
          .text("2018")
          .attr("x", xStart5)
          .attr("y", yStart5 - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");

        const radius6 = z(data[5]["Total Arrivals"] / 5000 - margin);
        const xStart6 = x(data[5]["Year"] + radius6 / 100 - 0.3);
        const yStart6 = height * 1.5 - radius6;
        const circle6 = g6
          .append("path")
          .attr("class", "second year6")
          .attr(
            "d",
            `M ${xStart6},${yStart6}
                a ${radius6} ${radius6} 0 1 1 0 ${radius6 * 2}
                  ${radius6} ${radius6} 0 1 1 0 ${-radius6 * 2} z`
          );
        g6.append("text")
          .text("2019")
          .attr("x", xStart6)
          .attr("y", yStart6 - 10)
          .attr("class", "text-years")
          .attr("font-family", "Courier")
          .attr("fill", "#2c3e50")
          .attr("font-weight", "bold")
          .style("opacity", "0")
          .attr("font-size", "0.8em")
          .attr("text-anchor", "middle");

        circle
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        circle2
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        circle3
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        circle4
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        circle5
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        circle6
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", "1.5")
          .attr("opacity", 0);

        var x = d3
          .scaleLinear()
          .domain([2014, 2020])
          .range([xStart - 100, xStart6 + 100]);
      });
      this.svg
        .append("image")
        .attr("class", "img")
        .attr(
          "xlink:href",
          "https://api.time.com/wp-content/uploads/2015/10/italy-migrants-refugees-asylum-seekers-1.jpg"
        )
        // .attr("height",this.height/1.5)
        // .attr("width",this.width /1.5)
        .style("opacity", 0);

      const projection = d3
        .geoMercator()
        .fitSize([2.4 * this.width, 2.4 * this.height], map_json);

      const path = d3.geoPath().projection(projection);

      let groups_map = this.svg
        .selectAll(".evros")
        .data(map_json.features)
        .enter()
        // .append("g")
        .filter(
          (d) =>
            d.properties.name === "Syria" ||
            d.properties.name === "Greece" ||
            d.properties.name === "Turkey"
        )
        .append("g");

      groups_map
        .append("path")
        .attr("d", path)
        .attr("class", "evros")
        .attr("fill", "#b35525")
        .attr("stroke", "black")
        .attr("opacity", "0");
      groups_map
        .append("text")
        .text((d) => d.properties.name)
        .attr("color", "black")
        .attr("class", "countries");

      groups_map.attr("transform", "translate(-1400,-300)");
      d3.selectAll(".evros")
        .append("text")
        .text((d) => d.properties.name)
        .attr("color", "black")
        .attr("class", "countries");

      // this.svg
      //   .append("text")
      //   .text(
      //     "The number of sea arrivals through the mediterranean increased significantly reaching 209,660 people"
      //   )
      //   .attr("transform", "translate(0,300)")
      //   .style("font-size", "20px")
      //   .attr("class", "first_text")
      //   .attr("opacity", "0");

      this.svg
        .append("defs")
        .append("g")
        .attr("id", "iconCustom")
        .append("path")
        .attr(
          "d",
          "M3.5,2H2.7C3,1.8,3.3,1.5,3.3,1.1c0-0.6-0.4-1-1-1c-0.6,0-1,0.4-1,1c0,0.4,0.2,0.7,0.6,0.9H1.1C0.7,2,0.4,2.3,0.4,2.6v1.9c0,0.3,0.3,0.6,0.6,0.6h0.2c0,0,0,0.1,0,0.1v1.9c0,0.3,0.2,0.6,0.3,0.6h1.3c0.2,0,0.3-0.3,0.3-0.6V5.3c0,0,0-0.1,0-0.1h0.2c0.3,0,0.6-0.3,0.6-0.6V2.6C4.1,2.3,3.8,2,3.5,2z"
        );

      // this.svg.append("rect").attr("width", 500).attr("height", 500).attr("top",500);

      //specify the number of columns and rows for pictogram layout
      var numCols = 10;
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
      // this.svg
      //   .append("text")
      //   .attr("id", "txtValue")
      //   .attr("x", xPadding)
      //   .attr("y", yPadding)
      //   .attr("dy", -3)
      //   .text("0");

      //create group element and create an svg <use> element for each icon
      this.svg
        .append("g")
        .attr("id", "pictoLayer")
        .selectAll("use")
        .data(myIndex)
        .enter()
        .append("use")
        .attr("xlink:href", "#iconCustom")
        .attr("id", function (d) {
          return "icon" + d;
        })
        .attr("x", function (d) {
          var remainder = d % numCols; //calculates the x position (column number) using modulus
          return xPadding + remainder * wBuffer; //apply the buffer and return value
        })
        .attr("y", function (d) {
          var whole = Math.floor(d / numCols); //calculates the y position (row number)
          return yPadding + whole * hBuffer; //apply the buffer and return the value
        })

        .classed("iconPlain", true)
        .attr("opacity", "0");

      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".iconPlain").attr("transform", d3.event.transform);
      }

      d3.selectAll(".route1")
        .transition()
        .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity.translate(this.width / 2.4, 15).scale(3.5)
        );
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

      console.log(this.svg);
      console.log(d3.selectAll(".country"));

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

      console.log(pointsLine4);

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

    hideLegend: function () {
      d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 0);
      d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 0);
    },

    hideBubbles: function () {
      d3.selectAll(".bubbles").transition().duration(1000).style("opacity", 0);
      // d3.selectAll(".text-years")
      //   .transition()
      //   .duration(1000)
      //   .style("opacity", 1);
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
      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Tunisia" || d.properties.name === "Libya"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#800020")
      //   .attr("fill-opacity", "0.5");

      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "Italy")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#006b80")
      //   .attr("fill-opacity", "0.5");
      // console.log(d3.select("mylabels"));

      // d3.selectAll(".mylabels").transition().duration(1000).attr("opacity", 1);
      // d3.selectAll(".mydots").transition().duration(1000).attr("opacity", 1);

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
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-600, -100).scale(1.8));

      d3.selectAll(".route1")
        .transition()
        .duration(750)
        .call(
          zoomLines1.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );
      d3.selectAll(".route2")
        .transition()
        .duration(750)
        .call(
          zoomLines2.transform,
          d3.zoomIdentity.translate(-600, -100).scale(1.8)
        );
      d3.selectAll(".route3")
        .transition()
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
      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }

      d3.selectAll(".country")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-400, -400).scale(1.7));

      var zoomLines3 = d3.zoom().on("zoom", zoomedLines3);

      function zoomedLines3() {
        d3.selectAll(".route3").attr("transform", d3.event.transform);
      }

      d3.selectAll(".route3")
        .transition()
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

      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Tunisia" ||
      //       d.properties.name === "Libya" ||
      //       d.properties.name === "Italy"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#c4c1b6");

      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "Morocco")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#800020")
      //   .attr("fill-opacity", "0.5");
      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "Spain")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#006b80")
      //   .attr("fill-opacity", "0.5");

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
    },
    colorWestern2: function () {
      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Morocco" || d.properties.name === "Spain"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#c4c1b6");
      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Mauritania" ||
      //       d.properties.name === "Senegal"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#800020")
      //   .attr("fill-opacity", "0.5");
      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "the Canary Islands")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#006b80")
      //   .attr("fill-opacity", "0.5");

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
      // console.log("coloring central");
      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Morocco" || d.properties.name === "Spain"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#c4c1b6");
      // d3.selectAll(".country")
      //   .filter(
      //     (d) =>
      //       d.properties.name === "Mauritania" ||
      //       d.properties.name === "Senegal"
      //   )
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#c4c1b6");
      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "Turkey")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#800020")
      //   .attr("fill-opacity", "0.5");
      // d3.selectAll(".country")
      //   .filter((d) => d.properties.name === "Greece")
      //   .transition()
      //   .duration(1500)
      //   .attr("fill", "#006b80")
      //   .attr("fill-opacity", "0.5");
      //   .filter())

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
        .duration(750)
        .call(
          zoom.transform,
          d3.zoomIdentity.translate(-1000, -200).scale(1.9)
        );

      console.log("route 6 is ", d3.selectAll(".route6"));
      d3.selectAll(".route6")
        .transition()
        .duration(750)
        .call(
          zoomLines6.transform,
          d3.zoomIdentity.translate(-1000, -200).scale(1.9)
        );

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
    },
    firstImage: function () {
      d3.selectAll(".img")
        .transition()
        .duration(500)
        .ease(d3.easeLinear)
        .style("opacity", 1)
        .attr("height", this.height / 1.5)
        .attr("width", this.width / 1.5);
    },

    drawBarChart: function () {
      let pathLength2;
      d3.selectAll(".second")
        .attr("stroke-dasharray", function () {
          pathLength2 = 40 * this.getTotalLength();
        })
        .attr("opacity", 1)
        .attr("stroke-dasharray", pathLength2 + " " + pathLength2)
        .attr("stroke-dashoffset", pathLength2)
        .transition()
        .duration(1500)
        .ease(d3.easeLinear)
        .attr("stroke-dashoffset", 0);

      d3.selectAll(".text-years").style("opacity", "1").transition();
    },

    hideChart: function () {
      d3.selectAll(".second").attr("opacity", "0").transition();
      d3.selectAll(".text-years").style("opacity", "0").transition();
    },

    unzoomChart: function () {
      function zoomed() {
        d3.selectAll(".second").attr("transform", d3.event.transform);
      }
      function zoomedText() {
        d3.selectAll(".text-years").attr("transform", d3.event.transform);
      }

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomText = d3.zoom().on("zoom", zoomedText);

      d3.selectAll(".second")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(600, -50).scale(0.5));

      d3.selectAll(".text-years")
        .transition()
        .duration(750)
        .call(
          zoomText.transform,
          d3.zoomIdentity.translate(600, -50).scale(0.5)
        );
    },

    zoomBackChart: function () {
      function zoomed() {
        d3.selectAll(".second").attr("transform", d3.event.transform);
      }
      function zoomedText() {
        d3.selectAll(".text-years").attr("transform", d3.event.transform);
      }

      var zoom = d3.zoom().on("zoom", zoomed);
      var zoomText = d3.zoom().on("zoom", zoomedText);

      d3.selectAll(".second")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-600, 50).scale(1));

      d3.selectAll(".text-years")
        .transition()
        .duration(750)
        .call(zoomText.transform, d3.zoomIdentity.translate(-600, 50).scale(1));
    },

    resetChart: function () {
      d3.selectAll(".bubbles")
        .transition()
        .duration(1000)
        .style("fill", "#53292a")
        .style("fill-opacity", 0.5)
        .attr("stroke", "#2c3e50");
    },

    highlight2014: function () {
      d3.selectAll(".year1").attr("fill", "#766F81").transition();
      d3.selectAll("use").attr("class", function (d, i) {
        if (d < 11) {
          return "iconSelected";
        } else {
          return "iconPlain";
        }
      });
      // d3.selectAll(".evros").transition().delay(500).attr("opacity", "1");
      // d3.selectAll(".first_text").transition().delay(500).attr("opacity", "1");
    },

    highlight2015: function () {
      d3.selectAll(".year2").attr("fill", "#766F81").transition();
      d3.selectAll(".first_text").attr("opacity", "0");
      d3.selectAll(".evros").transition().delay(500).attr("opacity", "0");
      d3.selectAll(".first_text").transition().delay(500).attr("opacity", "0");
    },

    highlight2016: function () {
      d3.selectAll(".year3").attr("fill", "#766F81").transition();
    },

    highlight2017: function () {
      d3.selectAll(".year4").attr("fill", "#766F81").transition();
    },

    highlight2018: function () {
      d3.selectAll(".year5").attr("fill", "#766F81").transition();
    },
    highlight2019: function () {
      d3.selectAll(".year6").attr("fill", "#766F81").transition();
    },
    lastView: function (){
      d3.selectAll(".last")
        .attr("opacity","1")
    },

    removeHighlight: function () {
      d3.selectAll(".year1").attr("fill", "none");
      d3.selectAll(".year2").attr("fill", "none");
      d3.selectAll(".year3").attr("fill", "none");
      d3.selectAll(".year4").attr("fill", "none");
      d3.selectAll(".year5").attr("fill", "none");
      d3.selectAll(".year6").attr("fill", "none");
      d3.selectAll(".evros").attr("opacity", "0");
      d3.selectAll(".iconPlain").attr("opacity", "0");
      d3.selectAll(".iconSelected").transition().attr("opacity", "0");
    },

    resetZoom: function () {
      var zoom = d3.zoom().on("zoom", zoomed);

      function zoomed() {
        d3.selectAll(".country").attr("transform", d3.event.transform);
      }

      console.log(d3.selectAll(".country"));
      d3.selectAll(".country")
        .transition()
        .duration(750)
        .call(zoom.transform, d3.zoomIdentity.translate(-1000, -900));
    },

    showIcons: function () {
      d3.selectAll(".iconPlain").attr("opacity", "1");
    },
    hideRoute12: function () {
      d3.selectAll(".route1").transition().duration(200).attr("opacity", 0);
      d3.selectAll(".route2").transition().duration(200).attr("opacity", 0);
    },
    hideRoute3: function () {
      d3.selectAll(".route3").transition().duration(200).attr("opacity", 0);
    },
    hideRoute45: function () {
      d3.selectAll(".route4").transition().duration(200).attr("opacity", 0);
      d3.selectAll(".route5").transition().duration(200).attr("opacity", 0);
    },
    hideRoute6: function () {
      d3.selectAll(".route6").transition().duration(200).attr("opacity", 0);
    },
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
  padding: 2px;
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

text {
  font-family: "Courier New", Courier, monospace;
  /* font-weight: 800; */
  line-height: 1.4em;
  font-size: 1em;
}

p,
.countries {
  font-family: "Courier New", Courier, monospace;
  /* font-weight: 800; */
  line-height: 1.4em;
  font-size: 1em;
  opacity: 0.6;
}

.text-years {
  font-weight: 800;
  font-size: 30px;
  color: #2c3e50;
  opacity: 0.6;
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
#graphic,
#graphic2,
#graphic3 {
  /* margin: auto; */
  width: 100rem;
  flex-direction: row;
  align-items: top;
  justify-content: space-around;
  padding: 2px;
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
  /* display: inline-block; */
  position: fixed;
  top: 1rem;
  right: -50rem;
  /* right: -750px; */
  z-index: 1;
  /* margin-left: 0; */
  /* height: 100rem; */
  /* width: 1000rem; */
  padding: 2px;
}

#img {
  /* display: inline-block; */
  /* position: fixed; */
  /* top: 1rem; */

  left: 500px;
  z-index: 1;
  /* margin-left: 0; */
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

.text-years {
  font-weight: 800;
  font-size: 30px;
  color: #2c3e50;
  opacity: 0.6;
}

.scroll-container {
  height: 100vh;
  /* overflow-y: scroll; */
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
  height: 70px;
  justify-content: space-between;
  padding: 0 20px;
  position: fixed;
  transition: transform 300ms ease-in-out;
  transform: translateY(0);
  width: 96%;
  z-index: 10;
}

.header_title {
  display: inline-block;
  font-size: 20px;
  font-weight: 400;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.header_nav {
  list-style: none;
  padding: 0;
}

.header_navItem {
  display: inline-block;
  font-size: 15px;
  letter-spacing: 0.1em;
  margin-left: 20px;
  text-transform: uppercase;
  color: #dbd9d2;
}

.header_navItem a {
  text-decoration: none;
  color: #dbd9d2;
}

.header_navItem a:after {
  background-color: #dbd9d2;
  content: "";
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

text {
  fill: #bb6d82;
  text-anchor: left;
  font-size: 12px;
  font-family: sans-serif, Helvetica, Arial;
  font-weight: bold;
}

.iconPlain {
  fill: #a7a59b;
}

.iconSelected {
  fill: #bb6d82;
}

rect {
  fill: #fff1e0;
}
</style>
