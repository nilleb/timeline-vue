<script>
import Vue from "vue";
import TimelineVue from "@/TimelineView.vue";

import events from "@/../data/events.json";

export default Vue.extend({
  name: "ServeDev",
  data: function () {
    console.log(events);
    return { selectedProject: null, events: events, attributeName: "members" };
  },
  components: {
    TimelineVue,
  },
});
</script>

<template>
  <div id="app">
    <h1>Timeline view (per developer)</h1>
    <timeline-vue
      :period="'Quarter'"
      :events="events"
      :attributeName="attributeName"
      :selected="selectedProject && selectedProject.id"
    />
  </div>
</template>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
}
</style>