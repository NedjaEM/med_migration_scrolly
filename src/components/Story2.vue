<template>
  <div>
    <section class="step" v-for="step in steps" :key="step.id">
      <h1>{{ step.title }}</h1>
      <p>{{ step.description }}</p>
      <!-- <p>{{ active_index }}</p>
      <p>{{ step.id }}</p> -->
    </section>
  </div>
</template>

<script>
/* eslint-disable */
import * as d3 from "../../../lib/d3";

export default {
  name: "App",
  components: {},
  props: ["state", "active_index"],
  data: function () {
    return {
      steps: [
        {
          title: "",
          description:
            "Let's look into the numbers of arrivals through the Mediterrenean to Europe",
        },
         {
          title: "2014",
          description:
            "  ",
        },
        {
          title: "2015",
          description:
            "  ",
        },
        {
          title: "2016",
          description:
            "  ",
        },
        {
          title: "2017",
          description:
            "  ",
        },
        {
          title: "2018",
          description:
            "  ",
        },
        {
          title: "2019",
          description:
            "  ",
        }
      ],
    };
  },
  mounted() {
    console.log("state from story " + this.state);
  },
  method: {},
};
</script>

<style scoped>
p {
  font-family: "Courier New", Courier, monospace;
  font-weight: 800;
  line-height: 1.4em;
  font-size: 1em;
}

h1 {
  opacity: 0.8;
   font-family: "Courier New", Courier, monospace;
  /* font-weight: 800;
  line-height: 1.4em; */
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
</style>