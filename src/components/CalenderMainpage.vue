<template>
  <div class="calendar">
    <div class="controls">
      <button @click="prevMonth">Prev</button>
      <h2>{{ formattedMonth }}</h2>
      <button @click="nextMonth">Next</button>
    </div>
    <div class="days-header">
      <div class="day-name" v-for="day in weekdays" :key="day">{{ day }}</div>
    </div>
    <div class="days-grid">
      <div class="day" v-for="(day, index) in days" :key="index">
        {{ day }}
      </div>
    </div>
  </div>
</template>

<script>
import { format, subMonths, addMonths, startOfWeek, endOfWeek, startOfMonth, endOfMonth, eachDayOfInterval } from 'date-fns';

export default {
  data() {
    return {
      currentDate: new Date(),
      availability: {
        '2024-08-01': true,
        '2024-08-05': false,
        '2024-08-15': true,
        '2024-08-20': false,
        // Add more dates as needed
      },
    };
  },
  computed: {
    formattedMonth() {
      return format(this.currentDate, 'MMMM yyyy');
    },
    days() {
      const startDay = startOfWeek(startOfMonth(this.currentDate));
      const endDay = endOfWeek(endOfMonth(this.currentDate));
      return eachDayOfInterval({ start: startDay, end: endDay }).map(day => day.getDate());
    },
    weekdays() {
      return ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
    },
    computed: {
      days() {
        const startDay = startOfWeek(startOfMonth(this.currentDate));
        const endDay = endOfWeek(endOfMonth(this.currentDate));
        return eachDayOfInterval({ start: startDay, end: endDay }).map(day => ({
          date: day.getDate(),
          isAvailable: this.availability[format(day, 'yyyy-MM-dd')] // Check availability
        }));
      },
    },
  },
  methods: {
    prevMonth() {
      this.currentDate = subMonths(this.currentDate, 1);
    },
    nextMonth() {
      this.currentDate = addMonths(this.currentDate, 1);
    },
  },
};
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 600px;
  margin: 0 auto;
}
.controls {
  display: flex;
  justify-content: space-between;
  width: 100%;
}
.days-header, .days-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
}
.day-name, .day {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
}
</style>
