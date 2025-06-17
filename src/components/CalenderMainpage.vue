<template>
  <div class="calendar">
    <!-- Party size selector -->
    <div class="party-size-selector">
      <label for="party-size">Party size:</label>
      <select id="party-size" v-model="partySize" @change="fetchData">
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

    <!-- Calendar controls -->
    <div class="controls">
      <button @click="changeMonth(-1)" aria-label="Previous month">Prev</button>
      <h2>{{ formattedMonth }}</h2>
      <button @click="changeMonth(1)" aria-label="Next month">Next</button>
    </div>

    <!-- Calendar grid -->
    <div class="days-header">
      <div class="day-name" v-for="day in weekdays" :key="day">{{ day }}</div>
    </div>
    <div class="days-grid">
      <div
          v-for="day in days"
          :key="day.date"
          :class="['day', {
          'available': isAvailable(day.date),
          'unavailable': !isAvailable(day.date) && isScheduledDate(day.date),
          'no-tour': !isScheduledDate(day.date),
          'selected': selectedDate && isSameDay(selectedDate, day.date)
        }]"
          @click="selectDate(day.date)"
      >
        {{ day.dayOfMonth }}
      </div>
    </div>

    <!-- Capacity info panel -->
    <div v-if="selectedDateInfo" class="capacity-info">
      <h3>{{ selectedDateInfo.tourName }}</h3>
      <p><strong>Date:</strong> {{ format(selectedDate, 'MMMM d, yyyy') }}</p>
      <div class="capacity-bar">
        <div class="capacity-fill" :style="{ width: capacityPercentage + '%' }"></div>
        <span class="capacity-text">{{ selectedDateInfo.currentBookings }}/{{ selectedDateInfo.maxCapacity }} people</span>
      </div>
      <p><strong>Available spots:</strong> {{ selectedDateInfo.availableSpots }}</p>
      <p><strong>Your party size:</strong> {{ partySize }} people</p>

      <div class="booking-status">
        <div v-if="selectedDateInfo.canAccommodateParty" class="available-message">
          ✅ Available for your party of {{ partySize }}
          <button class="book-button" @click="startBooking">Book Now</button>
        </div>
        <div v-else class="unavailable-message">
          ❌ Not enough space for your party of {{ partySize }}
        </div>
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="isLoading" class="loading">
      Loading availability...
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
      partySize: 2,
      weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      availableDateTimes: [],
      scheduledTours: [],
      selectedDate: null,
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
    },
    selectedDateInfo() {
      if (!this.selectedDate) return null;

      const tour = this.scheduledTours.find(tour =>
          isSameDay(parseISO(tour.localDate), this.selectedDate)
      );

      if (!tour) return null;

      const currentBookings = tour.currentBookings || 0;
      const maxCapacity = tour.tour.maxCapacity;
      const availableSpots = maxCapacity - currentBookings;
      const canAccommodateParty = availableSpots >= this.partySize;

      return {
        tourName: tour.tour.name,
        currentBookings,
        maxCapacity,
        availableSpots,
        canAccommodateParty
      };
    },
    capacityPercentage() {
      if (!this.selectedDateInfo) return 0;
      return (this.selectedDateInfo.currentBookings / this.selectedDateInfo.maxCapacity) * 100;
    }
  },
  methods: {
    async fetchData() {
      this.isLoading = true;
      try {
        const year = this.currentDate.getFullYear();
        const month = this.currentDate.getMonth() + 1;

        // Fetch available dates for current party size
        const availabilityResponse = await axios.get(`/api/tour-availability`, {
          params: { year, month, partySize: this.partySize }
        });

        this.availableDateTimes = availabilityResponse.data.map(dateStr => ({
          date: parseISO(dateStr),
          id: dateStr
        }));

        // Fetch all scheduled tours with capacity info
        const toursResponse = await axios.get('/api/scheduled-tours', {
          params: { year, month }
        });

        this.scheduledTours = toursResponse.data;

      } catch (error) {
        console.error('Error fetching data:', error);
      } finally {
        this.isLoading = false;
      }
    },

    selectDate(date) {
      if (!this.isScheduledDate(date)) return;
      this.selectedDate = date;
    },

    startBooking() {
      alert(`Starting booking for ${this.partySize} people on ${format(this.selectedDate, 'MMMM d, yyyy')}`);
    },

    changeMonth(delta) {
      this.currentDate = addMonths(this.currentDate, delta);
      this.selectedDate = null;
      this.fetchData();
    },

    isAvailable(date) {
      return this.availableDateTimes.some(dateTime =>
          isSameDay(dateTime.date, date)
      );
    },

    isScheduledDate(date) {
      return this.scheduledTours.some(tour =>
          isSameDay(parseISO(tour.localDate), date)
      );
    },

    isSameDay,
    format
  },
  created() {
    this.fetchData();
  }
};
</script>

<style scoped>

.day.no-tour {
  background-color: #f5f5f5;
  color: #ccc;
  cursor: not-allowed;
}

.capacity-bar {
  position: relative;
  width: 100%;
  height: 30px;
  background-color: #e0e0e0;
  border-radius: 15px;
  margin: 10px 0;
  overflow: hidden;
}

.capacity-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50 0%, #FFC107 70%, #f44336 90%);
  transition: width 0.3s ease;
}

.capacity-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: bold;
  color: #333;
  text-shadow: 1px 1px 2px rgba(255,255,255,0.8);
}

/* Rest of existing styles... */
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