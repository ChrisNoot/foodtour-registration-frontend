<template>
  <div class="calendar">

    <!-- Party size selector with 1-10 options -->
    <div class="party-size-selector">
      <label for="party-size">Party size:</label>
      <select id="party-size" v-model="partySize" @change="fetchAvailableDates">
        <option value="1">1 person</option>
        <option value="2">2 people</option>
        <option value="3">3 people</option>
        <option value="4">4 people</option>
        <option value="5">5 people</option>
        <option value="6">6 people</option>
        <option value="7">7 people</option>
        <option value="8">8 people</option>
        <option value="9">9 people</option>
        <option value="10">10 people</option>
      </select>
    </div>

    <div class="controls">
      <button @click="changeMonth(-1)" aria-label="Previous month">Prev</button>
      <h2>{{ formattedMonth }}</h2>
      <button @click="changeMonth(1)" aria-label="Next month">Next</button>
    </div>
    <div class="days-header">
      <div class="day-name" v-for="day in weekdays" :key="day">{{ day }}</div>
    </div>
    <div class="days-grid">
      <div
          v-for="day in days"
          :key="day.date"
          :class="['day', { 'available': isAvailable(day.date), 'unavailable': !isAvailable(day.date) }]"
          @click="selectDate(day.date)"
      >
        {{ day.dayOfMonth }}
      </div>
    </div>
    <div v-if="selectedDateTimes.length > 0" class="time-slots">
      <h3>Available times for {{ format(selectedDate, 'MMMM d, yyyy') }}:</h3>
      <button
          v-for="dateTime in selectedDateTimes"
          :key="dateTime.id"
          @click="selectDateTime(dateTime)"
      >
        {{ format(new Date(dateTime.time), 'h:mm a') }}
      </button>
    </div>
  </div>
</template>

<script>
import {
  format,
  addMonths,
  startOfWeek,
  endOfWeek,
  startOfMonth,
  endOfMonth,
  eachDayOfInterval,
  isSameDay,
  parseISO
} from 'date-fns';
import axios from 'axios';

export default {
  name: 'CalendarMainpage',
  data() {
    return {
      currentDate: new Date(),
      partySize: 2, // Default to 2 people
      weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      availableDateTimes: [],
      selectedDate: null,
      selectedDateTimes: [],
      isLoading: false
    };
  },
  computed: {
    formattedMonth() {
      return format(this.currentDate, 'MMMM yyyy');
    },
    days() {
      const startDay = startOfWeek(startOfMonth(this.currentDate));
      const endDay = endOfWeek(endOfMonth(this.currentDate));
      return eachDayOfInterval({ start: startDay, end: endDay })
          .map(date => ({
            date,
            dayOfMonth: date.getDate()
          }));
    }
  },
  methods: {
    async fetchAvailableDates() {
      this.isLoading = true;
      try {
        const year = this.currentDate.getFullYear();
        const month = this.currentDate.getMonth() + 1; // JS months are 0-based

        const response = await axios.get(`/api/tour-availability`, {
          params: {
            year,
            month,
            partySize: this.partySize
          }
        });

        this.availableDateTimes = response.data.map(dateStr => ({
          date: parseISO(dateStr),
          id: dateStr
        }));
      } catch (error) {
        console.error('Error fetching available dates:', error);
      } finally {
        this.isLoading = false;
      }
    },
    changeMonth(delta) {
      this.currentDate = addMonths(this.currentDate, delta);
    },
    isAvailable(date) {
      return this.availableDateTimes.some(dateTime =>
          isSameDay(dateTime.date, date)
      );
    },
    selectDate(date) {
      if (this.isAvailable(date)) {
        this.selectedDate = date;
        this.selectedDateTimes = this.availableDateTimes.filter(dateTime =>
            isSameDay(dateTime.date, date)
        );
      }
    },
    selectDateTime(dateTime) {
      this.$emit('datetime-selected', dateTime);
    }
  },
  created() {
    this.fetchAvailableDates();
  }
};
</script>

<style scoped>
.party-size-selector {
  margin-bottom: 1rem;
  text-align: center;
}

.party-size-selector label {
  margin-right: 0.5rem;
  font-weight: bold;
}

.party-size-selector select {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
}

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
  margin-bottom: 1rem;
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

.day {
  cursor: pointer;
}

.day.available {
  background-color: #e6f7e6;
}

.day.unavailable {
  background-color: #f7e6e6;
  cursor: not-allowed;
}

.time-slots {
  margin-top: 1rem;
  text-align: center;
}

.time-slots button {
  margin: 0.5rem;
  padding: 0.5rem 1rem;
  background-color: #e6f7e6;
  border: 1px solid #4CAF50;
  border-radius: 4px;
  cursor: pointer;
}

.time-slots button:hover {
  background-color: #4CAF50;
  color: white;
}
</style>