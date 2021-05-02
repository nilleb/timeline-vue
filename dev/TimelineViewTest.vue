<template>
  <div>
    <h2>Quarter, no resource</h2>
    <timeline-view
      :events="quarterEvents"
      :period="'quarter'"
    />
    <h2>Quarter, by resource</h2>
    <timeline-view
      :events="quarterEvents"
      :attributeName="'owner'"
      :period="'quarter'"
    />
    <h2>Week, by resource</h2>
    <timeline-view
      :events="weekEvents"
      :attributeName="'owner'"
      :period="'week'"
    />
    <h2>Day, by resource</h2>
    <timeline-view
      :events="dayEvents"
      :attributeName="'owner'"
      :period="'day'"
    />
  </div>
</template>

<script>
import dayjs from "dayjs";
var weekOfYear = require("dayjs/plugin/weekOfYear");
dayjs.extend(weekOfYear);
var weekday = require("dayjs/plugin/weekday");
dayjs.extend(weekday);
var quarterOfYear = require("dayjs/plugin/quarterOfYear");
dayjs.extend(quarterOfYear);
import TimelineView from "./TimelineView.vue";

export default {
  components: { TimelineView },
  data() {
    return {
      quarterEvents: [],
      weekEvents: [],
      dayEvents: [],
    };
  },
  setup() {},
  methods: {},
  computed: {},
  mounted() {
    const firstDayOfQuarter = dayjs().startOf("quarter").startOf("day");
    const today = dayjs().startOf("day");
    this.quarterEvents = [
      {
        name: "one",
        startDate: firstDayOfQuarter,
        endDate: firstDayOfQuarter.add(4, "week"),
        owner: "John",
      },
      {
        name: "two",
        startDate: firstDayOfQuarter.add(4, "week"),
        endDate: firstDayOfQuarter.add(7, "week"),
        owner: "John",
      },
      {
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "week"),
        owner: "Paul",
      },
      {
        name: "other",
        startDate: today.add(1, "week"),
        endDate: today.add(4, "week"),
        owner: "Jack",
      },
    ];
    this.weekEvents = [
      {
        name: "one",
        startDate: today,
        endDate: today.add(4, "day"),
        owner: "John",
      },
      {
        name: "two",
        startDate: today.add(4, "day"),
        endDate: today.add(7, "day"),
        owner: "John",
      },
      {
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "day"),
        owner: "Paul",
      },
      {
        name: "other",
        startDate: today.add(1, "day"),
        endDate: today.add(4, "day"),
        owner: "Jack",
      },
    ];
    this.dayEvents = [
      {
        name: "one",
        startDate: today,
        endDate: today.add(4, "hour"),
        owner: "John",
      },
      {
        name: "two",
        startDate: today.add(4, "hour"),
        endDate: today.add(7, "hour"),
        owner: "John",
      },
      {
        name: "first",
        startDate: dayjs(),
        endDate: today.add(4, "hour"),
        owner: "Paul",
      },
      {
        name: "other",
        startDate: today.add(1, "hour"),
        endDate: today.add(4, "hour"),
        owner: "Jack",
      },
    ];
  },
};
</script>
