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
const locales = getBrowserLocales({ languageCodeOnly: true });
const locale = locales ? locales[0] : "en";
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
    return {
      selectedProject: null,
      events: events,
      attributeName: "members",
      quarterEvents: [],
      weekEvents: [],
      dayEvents: [],
    };
  },
  components: {
    TimelineVue,
  },
  mounted() {
    const firstDayOfQuarter = dayjs().startOf("quarter").startOf("day");
    const today = dayjs().startOf("day");
    this.quarterEvents = [
      {
        id: "qone",
        name: "one",
        startDate: firstDayOfQuarter,
        endDate: firstDayOfQuarter.add(4, "week"),
        owner: "John",
      },
      {
        id: "qtwo",
        name: "two",
        startDate: firstDayOfQuarter.add(4, "week"),
        endDate: firstDayOfQuarter.add(7, "week"),
        owner: "John",
      },
      {
        id: "qfirst",
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "week"),
        owner: "Paul",
      },
      {
        id: "qother",
        name: "other",
        startDate: today.add(1, "week"),
        endDate: today.add(4, "week"),
        owner: "Jack",
      },
    ];
    this.weekEvents = [
      {
        id: "wone",
        name: "one",
        startDate: today,
        endDate: today.add(4, "day"),
        owner: "John",
      },
      {
        id: "wtwo",
        name: "two",
        startDate: today.add(4, "day"),
        endDate: today.add(7, "day"),
        owner: "John",
      },
      {
        id: "wfirst",
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "day"),
        owner: "Paul",
      },
      {
        id: "wother",
        name: "other",
        startDate: today.add(1, "day"),
        endDate: today.add(4, "day"),
        owner: "Jack",
      },
    ];
    this.dayEvents = [
      {
        id: "done",
        name: "one",
        startDate: today,
        endDate: today.add(4, "hour"),
        owner: "John",
      },
      {
        id: "dtwo",
        name: "two",
        startDate: today.add(4, "hour"),
        endDate: today.add(7, "hour"),
        owner: "John",
      },
      {
        id: "dfirst",
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "hour"),
        owner: "Paul",
      },
      {
        id: "dother",
        name: "other",
        startDate: today.add(1, "hour"),
        endDate: today.add(4, "hour"),
        owner: "Jack",
      },
    ];
  },
});
</script>

<template>
  <div id="app">
    <h1>Timeline view (per developer)</h1>
    <ul style="list-style-type: none">
      <li v-for="event in events" :key="event.id">
        {{ event.name }} - {{ event.startDate }} - {{ event.endDate }}
      </li>
    </ul>
    <p style="margin: 50px 0px;">
      Drag the items on the timeline below 👇 to see how the properties of the
      elements above ☝️ evolve.
    </p>
    <timeline-vue
      :period="'Quarter'"
      :events="events"
      :attributeName="attributeName"
      :selected="selectedProject && selectedProject.id"
    />
    <h2>Quarter, no resource</h2>
    <timeline-vue :events="quarterEvents" :period="'quarter'" />
    <h2>Quarter, by resource</h2>
    <timeline-vue
      :events="quarterEvents"
      :attributeName="'owner'"
      :period="'quarter'"
    />
    <h2>Week, by resource</h2>
    <timeline-vue
      :events="weekEvents"
      :attributeName="'owner'"
      :period="'week'"
    />
    <h2>Day, by resource</h2>
    <timeline-vue
      :events="dayEvents"
      :attributeName="'owner'"
      :period="'day'"
    />
  </div>
</template>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
}
</style>