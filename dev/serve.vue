<script>
import Vue from "vue";
import TimelineVue from "@/TimelineView.vue";
import dayjs from "dayjs";

import events from "@/../data/events.json";

function getBrowserLocales(options = {}) {
  const defaultOptions = {
    languageCodeOnly: false,
  };
  const opt = {
    ...defaultOptions,
    ...options,
  };
  const browserLocales =
    navigator.languages === undefined
      ? [navigator.language]
      : navigator.languages;
  if (!browserLocales) {
    return undefined;
  }
  return browserLocales.map((locale) => {
    const trimmedLocale = locale.trim();
    return opt.languageCodeOnly ? trimmedLocale.split(/-|_/)[0] : trimmedLocale;
  });
}
const locales = getBrowserLocales({languageCodeOnly: true})
const locale = locales ? locales[0] : 'en'
console.log(locale);
require(`dayjs/locale/en`);
require(`dayjs/locale/fr`);
require(`dayjs/locale/it`);
require(`dayjs/locale/es`);
dayjs.locale(locale);

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
    <ul>
      <li v-for="event in events" :key="event.id">{{ event.name }} - {{ event.startDate }} - {{ event.endDate }}</li>
    </ul>
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