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
  font-family: "Biryani";
  font-weight: 600;
  line-height: 1.4em;
  font-size: 1em;
   margin-top: -100%;
}

h1 {
  opacity: 0.8;
  font-family: "Khula";
  font-size: 1em;
   margin-bottom: 0px;

}

.step {
  height: 50rem;
  /* font-family: "Domine"; */
  font-family: "Biryani";
  font-weight: 400;
  line-height: 1.4em;
  font-size: 1.2em;
  text-align: left;;
  margin-left: 0.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

@media only screen and (max-width: 600px) {
  .step {
  height: 50rem;
  /* font-family: "Domine"; */
  font-family: "Biryani";
  font-weight: 400;
  line-height: 1.4em;
  font-size: 0.8em;
  text-align: left;;
  margin-left: 0.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}
}

</style>