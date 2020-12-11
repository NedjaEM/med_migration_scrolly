<template>
  <div>
    <section class="step" v-for="step in steps" :key="step.id">
      <h1>{{ step.title }}</h1>
      <p>{{ step.description }}</p>
      <div class="inst"> {{step.instructions}}</div>
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
            "According to data from the IOM (International Organization for Migration, almost 2 million people have arrived to Spain, Italy, Greece and Cyprus through the Mediterranean between 2014 and 2019.",
        },
        {
          title: "",
          description:
            "The 200 Km long Greek-Turkish border along the Evros river has been a common route for migrants coming from Syria, Iraq and Afghanistan\
            to cross to Greece from Turkey. However, in 2012, a tall barbed wire fence was build to block entry through the dry land \
            part of the border.",
        },
        {
          title: "",
          description:
            "This fence gave no other choice to migrants who were fleeing hard conditions in their countries but to seek other crossing alternatives\
            namely by crossing the mediterranean sea, a much more perilous route. The fence, instead of discouraging migrants to cross,\
            caused a high influx of people crossing through the Eastern Mediterranean route and putting their lives at risk.",
        },
        {
          title: "",
          description:
            "As you will see through the years, this pattern of attempting to curb numbers without addressing the root cause of why people are leaving, is a very\
            common response from European countries..",
        },
        {
          title: "",
          description:
            "According to a Human Rights Watch report, 85,500 people crossed in the first five months of 2015. The International\
            Organization for Migration has estimated at least 1,850 deaths in the Mediterranean sea in hose first fives months. That number doubled by the end of the year. ",
        },
        {
          title: "",
          description:
            "According to the UN refugee agency, UNHCR, over 60 percent of the people who crossed the mediterrenean in the first five months of 2015 came from Syria,\
             Somalia, and Afghanistan and Eritrea. All countries are in the middle of a war or are facing increased violence.",
        },
        {
          title: "",
          description:
            "Human Rights Watch interviewed over 150 migrants arriving to Italy and Greece. In the report, testimonies by migrants on the reasons why they have fled are described.\
            ",
          instructions:
          "Click on the icons to read testimonies of people from Syria, Somalia, Afghanistan and Eritrea "
        },
        {
          title: "",
          description:
            "5,143 people have drowned in 2016, the highest number ever recorded.\
             Since the counting of migrants only happens if they are intercepted by law enforcement and the counting of deaths only happebs if\
            bodies are found, these numbers are and will always be an underestimate of the actual number of lives lost in the mediterranean",
        },
        {
          title: "",
          description:
            "EU-Turkey Statement, another attempt at curbing numbers",
        },
        {
          title: "",
          description: " Italy-Libya Agreement",
        },
        {
          title: "",
          description:
            "Most migrants leaving libya to italy are not libyans. libya still attracts migrants from other countries despite the level of insecurity. Libya does not provide the status of refugee\
Some people entered libya to get to europe but ended up stuck in libya in detention centres",
        },
        {
          title: "",
          description: " Since 2017, as the number of migrants departing Libya has dropped, the number of people travelling from Tunisia to Italy has increased exponentially.",
        },
        {
          title: "",
          description: "  ",
        }
        //   {
        //   title: "",
        //   description: "  ",
        // }
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
  font-family: "Courier New", Courier;
  font-weight: 600;
  line-height: 1.4em;
  font-size: 1em;
   margin-top: -100%;
}

.inst {
   font-family: "Courier New", Courier;
  font-weight: 1000;
  line-height: 1.4em;
  font-size: 1em;
   margin-top: -100%;
}

h1 {
  opacity: 0.8;
  font-family: "Courier New", Courier;
  font-size: 1em;
   margin-bottom: 0px;

}

.step {
  height: 50rem;
  /* font-family: "Domine"; */
  font-family: "Courier New", Courier;
  font-weight: 400;
  line-height: 1.4em;
  font-size: 1.2em;
  text-align: left;;
  margin-left: 0.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}
</style>