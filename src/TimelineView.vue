<template>
  <div class="vue-scheduler">
    <div class="scheduler-container">
      <div
        class="timeline"
        :style="{ width: timelineWidth + 'px', height: timelineHeight + 'px' }"
      >
        <div class="resource-slot" v-if="attributeName">
          <b>Resource</b>
        </div>
        <div
          :class="`timeline-slot ${
            selectedSlots.includes(slot.id) ? 'selected' : ''
          }`"
          v-for="slot in slots"
          :key="slot.id"
          :style="{ width: timelineSlotWidth + 'px' }"
          :title="slot.tooltip"
          @click="onSelectSlot(slot.id)"
        >
          {{ slot.name }}
        </div>
      </div>

      <div class="resources">
        <div
          class="resource-row"
          v-for="(resource, i) in resourceLines"
          :key="i"
          :style="{ height: `${resource.height}px` }"
        >
          {{ resource.value }}
        </div>
      </div>

      <div class="events">
        <div class="event-row" v-for="(eventRow, i) in eventLines" :key="i">
          <div
            :id="`event-container-${i}-${event.id}`"
            v-for="event in eventRow"
            :key="event.id"
            :title="getEventTooltip(event)"
            :class="`event ${selected === event.id ? 'selected' : ''}`"
            :style="{
              left: event.position + 'px',
              width: event.width + 'px',
              cursor: 'ew-resize',
              backgroundColor: 'lightgray',
            }"
            @mousedown="attachDrag(event, $event)"
          >
            <div
              :id="`event-cell-${i}-${event.id}`"
              :title="getEventTooltip(event)"
              :style="{
                width: '100%',
                cursor: 'move',
                backgroundColor: 'white',
                paddingTop: '8px',
                paddingBottom: '8px',
              }"
            >
              {{ `${event.name}` }}
            </div>
          </div>
        </div>
      </div>

      <div
        class="now"
        :style="{ left: nowMarkerPosition + 'px' }"
        ref="now"
      ></div>
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs";
import { v4 as uuidv4 } from "uuid";
var weekOfYear = require("dayjs/plugin/weekOfYear");
dayjs.extend(weekOfYear);
var weekday = require("dayjs/plugin/weekday");
dayjs.extend(weekday);
var quarterOfYear = require("dayjs/plugin/quarterOfYear");
dayjs.extend(quarterOfYear);

function isIterable(obj) {
  if (obj == null) {
    return false;
  }
  return typeof obj[Symbol.iterator] === "function";
}

const validatePeriod = function (value) {
  return ["quarter", "week", "day"].includes(value.toLowerCase());
};

const validateEvents = function (value) {
  return Boolean(
    value.filter(
      (item) =>
        item.id !== undefined &&
        item.name !== undefined &&
        item.startDate !== undefined &&
        item.endDate !== undefined
    )
  );
};

export default {
  setup() {},
  components: {},
  props: {
    period: {
      type: String,
      validator: validatePeriod,
    },
    events: {
      type: Array,
      validator: validateEvents,
    },
    attributeName: String,
    selected: String,
  },
  data() {
    return {
      timelineSlotWidth: 100,
      nowMarkerPosition: 0,
      selectedSlots: [],
    };
  },
  methods: {
    attachDrag(eventObject, mouseDownEvent) {
      mouseDownEvent.preventDefault();
      document.onmouseup = this.detachDrag(eventObject.id);
      document.onmousemove = this.dragElement(
        mouseDownEvent.clientX,
        eventObject
      );
    },
    dragElement(initialX, eventObject) {
      const event = this.events.find((event) => event.id == eventObject.id);
      const eventStartDate = dayjs(event.startDate);
      const eventEndDate = dayjs(event.endDate);
      const leftBorderDistance = Math.abs(initialX - eventObject.position);
      const rightBorderDistance = Math.abs(
        leftBorderDistance - eventObject.width
      );
      const what =
        leftBorderDistance < 16
          ? "left"
          : rightBorderDistance < 8
          ? "right"
          : "both";
      console.log(leftBorderDistance, rightBorderDistance, what);
      const that = this;
      const dragTo = function (moveEvent) {
        const gap = moveEvent.clientX - initialX;
        if (what === "left" || what === "both") {
          event.startDate = eventStartDate.add(
            gap / that.timelineSlotWidth,
            that.localUnit.toLowerCase()
          );
        }
        if (what === "right" || what === "both") {
          event.endDate = eventEndDate.add(
            gap / that.timelineSlotWidth,
            that.localUnit.toLowerCase()
          );
        }
      };
      return dragTo;
    },
    detachDrag(eventId) {
      const that = this;
      return function () {
        const event = that.events.find((event) => event.id == eventId);
        that.$emit("eventChanged", event);
        document.onmouseup = null;
        document.onmousemove = null;
      };
    },
    onSelectSlot(slotId) {
      console.log("select slot ", slotId, this.selectedSlots);
      const index = this.selectedSlots.indexOf(slotId);
      if (index > -1) {
        this.selectedSlots.splice(index, 1);
      } else {
        this.selectedSlots.push(slotId);
      }
      this.setToLocalStorage("selectedSlots", this.selectedSlots);
    },
    slotsCount() {
      if (this.periodName === "quarter") {
        return this.endOfPeriod.week() - this.beginningOfPeriod.week();
      } else if (this.periodName === "week") {
        return 7;
      }
      return 24;
    },
    totalSlots() {
      return this.slotsCount() + (this.attributeName ? 1 : 0);
    },
    getEventTooltip(event) {
      const format =
        this.periodName === "quarter" ? "MMM DD" : "H:mm A, MMM DD YYYY";
      return `${event.name} (${dayjs(dayjs(event.startDate)).format(
        format
      )} -> ${dayjs(dayjs(event.endDate)).format(format)})`;
    },
    overlaps(event, events) {
      const es = dayjs(event.startDate);
      const ee = dayjs(event.endDate);
      for (let other of events) {
        const os = dayjs(other.startDate);
        const oe = dayjs(other.endDate);
        if (!(ee < os || oe < es)) {
          return other;
        }
      }
      return false;
    },
    accumulate(r, element, a, that) {
      r[element] = r[element] || { lines: [] };
      let event = that.convertEvent(a);
      for (const line of r[element].lines) {
        if (line.events === undefined) {
          line.events = [event];
          event = null;
        } else if (!that.overlaps(a, line.events)) {
          line.events.push(event);
          event = null;
          break;
        }
      }
      if (event) {
        r[element].lines.push({ events: [event] });
      }
    },
    computeEventGrid(events, attributeName) {
      const that = this;
      if (events === undefined) return [];
      return events.reduce(function (r, a) {
        let value = a[attributeName];

        if (isIterable(value) && typeof value !== "string") {
          if (value.length == 0) {
            console.log(`⚠️ the item ${a.name} has no value`);
            value.push("non assigned");
          }
        } else {
          value = [value];
        }

        value.forEach((element) => that.accumulate(r, element, a, that));

        return r;
      }, Object.create(null));
    },
    convertEvent(event) {
      const { name, startDate, endDate, id } = event;
      const position = this.computePosition(startDate);
      return {
        name: name,
        position: position,
        startDate: startDate,
        endDate: endDate,
        width: this.computePosition(endDate) - position,
        id: id !== undefined ? id : uuidv4(),
      };
    },
    limit(instant) {
      if (instant < this.beginningOfPeriod) {
        return this.beginningOfPeriod;
      }
      if (instant > this.endOfPeriod) {
        return this.endOfPeriod;
      }
      return instant;
    },
    computePosition(date) {
      const evaluatedDate = dayjs(date);
      const period = this.periodName;
      const fixedMarginLeft =
        (this.attributeName ? 1 : 0) * this.timelineSlotWidth;
      if (period == "quarter") {
        const workDate = this.limit(evaluatedDate);
        return (
          fixedMarginLeft +
          (workDate.week() - this.beginningOfPeriod.week()) * this.timelineSlotWidth +
          (workDate.weekday() * this.timelineSlotWidth) / 7
        );
      } else if (period == "week") {
        return (
          fixedMarginLeft +
          evaluatedDate.day() * this.timelineSlotWidth +
          (evaluatedDate.hour() * this.timelineSlotWidth) / 60
        );
      } else if (period == "day") {
        return (
          fixedMarginLeft +
          evaluatedDate.hour() * this.timelineSlotWidth +
          (evaluatedDate.minute() * this.timelineSlotWidth) / 60
        );
      }
    },
    computeNowMarkerPosition() {
      return this.computePosition(dayjs());
    },
    followingMonday(day) {
      return day.add(7 - day.day() + 1, "day");
    },
    followingSunday(day) {
      return day.add(0 - day.day() + 1, "day");
    },
    setToLocalStorage(key, item) {
      localStorage.setItem(key, JSON.stringify(item));
    },
    retrieveFromLocalStorage(key) {
      if (localStorage.getItem(key)) {
        try {
          return JSON.parse(localStorage.getItem(key));
        } catch (e) {
          localStorage.removeItem(key);
        }
      }
    },
  },
  computed: {
    periodName: function () {
      return this.period.toLowerCase();
    },
    localUnit: function () {
      return this.periodName === "quarter"
        ? "Week"
        : this.periodName === "week"
        ? "Day"
        : "Hour";
    },
    beginningOfPeriod: function () {
      let now = dayjs();
      if (this.periodName === "quarter") {
        now = this.followingMonday(now.startOf("quarter"));
      } else if (this.periodName === "week") {
        now = now.startOf("week");
      } else {
        now = now.startOf("day");
      }
      return now;
    },
    endOfPeriod: function () {
      let now = dayjs();
      if (this.periodName === "quarter") {
        now = this.followingSunday(now.endOf("quarter"));
      } else if (this.periodName === "week") {
        now = now.endOf("week");
      } else {
        now = now.endOf("day");
      }
      return now;
    },
    slots: function () {
      var i;
      var data = [];
      if (this.periodName === "quarter") {
        const firstQuarterWeek = this.beginningOfPeriod.week();

        for (i = 0; i < this.slotsCount(); i++) {
          data.push({
            id: i,
            name: `${this.localUnit} ${i + firstQuarterWeek}`,
            tooltip: `${this.beginningOfPeriod
              .add(i, "week")
              .format("MMM DD")} -> ${this.beginningOfPeriod
              .add(i, "week")
              .add(6, "day")
              .format("MMM DD")}`,
          });
        }
      } else if (this.periodName === "week") {
        const firstDayOfWeek = dayjs().startOf("week");
        for (i = 0; i < this.slotsCount(); i++) {
          data.push({
            id: i,
            name: `${firstDayOfWeek.add(i, "day").format("MMM DD")}`,
          });
        }
      } else if (this.periodName === "day") {
        for (i = 0; i < this.slotsCount(); i++) {
          data.push({
            id: i,
            name: `${this.localUnit} ${i + 1}`,
          });
        }
      }
      return data;
    },
    eventGrid: function () {
      return this.computeEventGrid(this.events, this.attributeName);
    },
    resourceLines: function () {
      let resources = [];
      for (const [key, resource] of Object.entries(this.eventGrid)) {
        resource.value = key;
        resource.height = resource.lines.length * 46 - 10;
        resources.push(resource);
      }
      return resources;
    },
    eventLines: function () {
      let lines = [];
      for (const [, resource] of Object.entries(this.eventGrid)) {
        lines.push(...resource.lines.map((line) => line.events));
      }
      return lines;
    },
    timelineWidth: function () {
      return this.totalSlots() * this.timelineSlotWidth;
    },
    timelineHeight: function () {
      return (3 + this.eventLines.length) * 41;
    },
  },
  mounted() {
    this.selectedSlots = this.retrieveFromLocalStorage("selectedSlots") || [];
    this.timelineSlotWidth = 100;
    this.nowMarkerPosition = this.computeNowMarkerPosition();
  },
};
</script>

<style scoped>
.scheduler-container {
  width: 100%;
  border: 1px solid #ccc;
  overflow-x: scroll;
  position: relative;
}

.scheduler-container .timeline {
  height: 100%;
}

.scheduler-container .timeline .resource-slot {
  width: 100px;
  height: 100%;
  border-right: 1px solid #eee;
  box-sizing: border-box;
  float: left;
  background-color: #f2effb;
  color: #7957d5;
  font-size: 12px;
  text-indent: 5px;
  padding-top: 5px;
}

.scheduler-container .timeline .timeline-slot {
  width: 100px;
  height: 100%;
  border-right: 1px solid #eee;
  box-sizing: border-box;
  float: left;
  color: #aaa;
  font-size: 12px;
  text-indent: 5px;
  padding-top: 5px;
}

.scheduler-container .timeline .selected {
  background-color: #fffbeb;
}

.scheduler-container .now {
  border-left: 1px solid red;
  opacity: 0.5;
  height: 100%;
  position: absolute;
  top: 0px;
  z-index: 0;
}

.scheduler-container .resources {
  position: absolute;
  top: 20px;
  z-index: 1;
}

.scheduler-container .resources .resource-row {
  top: 20px;
  margin-bottom: 10px;
  color: #7957d5;
  box-shadow: 0 0 0.125em;
  position: relative;
  width: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  font-size: 12px;
}

.scheduler-container .events {
  position: absolute;
  top: 40px;
  z-index: 1;
}

.scheduler-container .events .event-row {
  margin-bottom: 10px;
  height: 36px;
  position: relative;
}

.scheduler-container .events .event {
  cursor: move;
  position: absolute;
  top: 0px;
  height: 36px;
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  padding: 0px 8px;
  display: flex;
  justify-content: center;
  font-size: 12px;
  box-shadow: 2px 2px 5px #ddd;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  z-index: 0;
}

.scheduler-container .events .selected {
  background-color: rgb(200, 240, 220);
  z-index: 1;
}
</style>
