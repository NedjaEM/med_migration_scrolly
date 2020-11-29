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
            "The 200 Km long Greek-Turkish border along the Evros river has been a common route for migrants coming from Syria, Iraq and Afghanistan\
            to cross to Greece from Turkey. However, in 2012, a tall barbed wire fence was build to block entry through the dry land \
            part of the border.",
        },
        {
          title: "2014",
          description:
            "This fence gave no other choice to migrants who were fleeing hard conditions in their countries but to seek other crossing alternatives\
            namely by crossing the mediterranean sea, a much more perilous route. The fence, instead of discouraging migrants to cross,\
            caused a high influx of people crossing through the Eastern Mediterranean route and putting their lives at risk.",
        },
        {
          title: "2014",
          description:
            "It is estimated that most people coming into Greece from Turkey qualify for refugee status in most European countries.",
        },
        {
          title: "2014",
          description:
            "As you will see through the years, this pattern of attempting to curb numbers without addressing the root cause of why people are leaving, is a very\
            common response from European countries..",
        },
        {
          title: "2015",
          description:
            "According to a Human Rights Watch report, 85,500 people crossed in the first five months of 2015. The International\
            Organization for Migration has estimated at least 1,850 deaths in the Mediterranean sea in hose first fives months. That number doubled by the end of the year. ",
        },
        {
          title: "2015",
          description:
            "According to the UN refugee agency, UNHCR, over 60 percent of the people who crossed the mediterrenean in the first five months of 2015 came from Syria,\
             Somalia, and Afghanistan and Eritrea. All countries are in the middle of a war or are facing increased violence.",
        },
        {
          title: "2015",
          description:
            "Human Rights Watch interviewed over 150 migrants arriving to Italy and Greece. In the report, testimonies by migrants on the reasons why they have fled are described.  ",
        },
        {
          title: "2015",
          description: "Syria",
        },
        {
          title: "2015",
          description: "Somalia",
        },
        {
          title: "2015",
          description: "Afghanistan",
        },
        {
          title: "2015",
          description: "Eritrea",
        },
        {
          title: "2016",
          description:
            "5,096 people have drowned in 2016, the highest number ever recorded.\
             Since the counting of migrants only happens if they are intercepted by law enforcement and the counting of deaths only happebs if\
            bodies are found, these numbers are and will always be an underestimate of the actual number of lives lost in the mediterranean",
        },
        {
          title: "2016",
          description:
            "EU-Turkey Statement, another attempt at curbing numbers",
        },
        {
          title: "2017",
          description: " Italy-Libya Agreement",
        },
        {
          title: "2017",
          description:
            "Most migrants leaving libya to italy are not libyans. libya still attracts migrants from other countries despite the level of insecurity. Libya does not provide the status of refugee\
Some people entered libya to get to europe but ended up stuck in libya in detention centres",
        },
        {
          title: "2018",
          description: " Since 2017, as the number of migrants departing Libya has dropped, the number of people travelling from Tunisia to Italy has increased exponentially.",
        },
        {
          title: "2019",
          description: "  ",
        },
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