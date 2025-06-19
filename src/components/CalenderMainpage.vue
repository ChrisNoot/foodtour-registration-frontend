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
      <div class="tour-header">
        <h3>{{ selectedDateInfo.tourName }}</h3>
        <p class="tour-date">{{ format(selectedDate, 'MMMM d, yyyy') }}</p>
      </div>

      <!-- Pricing section -->
      <div class="pricing-section">
        <div class="price-row">
          <span class="price-label">Price per person:</span>
          <span class="price-value">€{{ selectedDateInfo.pricePerPerson }}</span>
        </div>
        <div class="price-row">
          <span class="price-label">Party size:</span>
          <span class="price-value">{{ partySize }} {{ partySize === 1 ? 'person' : 'people' }}</span>
        </div>
        <div class="total-price-row">
          <span class="total-label">Total Price:</span>
          <span class="total-amount">€{{ totalPrice }}</span>
        </div>
      </div>

      <!-- Capacity section -->
      <div class="capacity-section">
        <div class="capacity-header">
          <span>Available spots: {{ selectedDateInfo.availableSpots }}</span>
        </div>
        <div class="capacity-bar">
          <div class="capacity-fill" :style="{ width: capacityPercentage + '%' }"></div>
          <span class="capacity-text">{{ selectedDateInfo.currentBookings }}/{{ selectedDateInfo.maxCapacity }} people</span>
        </div>
      </div>

      <!-- Booking status -->
      <div class="booking-section">
        <div v-if="selectedDateInfo.canAccommodateParty" class="booking-available">
          <div class="availability-badge">
            <span class="check-icon">✅</span>
            <span>Available for your party</span>
          </div>
          <button v-if="!showBookingForm" class="book-button" @click="showBookingForm = true">
            <span class="button-text">Book Now</span>
            <span class="button-price">€{{ totalPrice }}</span>
          </button>
        </div>
        <div v-else class="booking-unavailable">
          <div class="unavailability-badge">
            <span class="cross-icon">❌</span>
            <span>Not enough space for {{ partySize }} {{ partySize === 1 ? 'person' : 'people' }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Booking Form (slides down when Book Now is clicked) -->
    <transition name="slide-down">
      <div v-if="showBookingForm" class="booking-form">
        <div class="form-header">
          <h3>Complete Your Booking</h3>
          <button class="close-form" @click="closeBookingForm">×</button>
        </div>

        <div class="booking-summary-mini">
          <span>{{ selectedDateInfo.tourName }} • {{ format(selectedDate, 'MMM d, yyyy') }} • {{ partySize }} people • €{{ totalPrice }}</span>
        </div>

        <form @submit.prevent="processBooking" class="booking-details">
          <!-- Customer Information -->
          <div class="form-section">
            <h4>Your Details</h4>
            <div class="form-row">
              <div class="form-group">
                <label for="firstName">First Name *</label>
                <input
                    type="text"
                    id="firstName"
                    v-model="bookingForm.firstName"
                    required
                    placeholder="John"
                >
              </div>
              <div class="form-group">
                <label for="lastName">Last Name *</label>
                <input
                    type="text"
                    id="lastName"
                    v-model="bookingForm.lastName"
                    required
                    placeholder="Doe"
                >
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label for="email">Email *</label>
                <input
                    type="email"
                    id="email"
                    v-model="bookingForm.email"
                    required
                    placeholder="john@example.com"
                >
              </div>
              <div class="form-group">
                <label for="phone">Phone *</label>
                <input
                    type="tel"
                    id="phone"
                    v-model="bookingForm.phone"
                    required
                    placeholder="+31 6 1234 5678"
                >
              </div>
            </div>
          </div>

          <!-- Additional Information -->
          <div class="form-section">
            <h4>Additional Information <span class="optional">(Optional)</span></h4>
            <div class="form-group">
              <label for="dietary">Dietary Restrictions or Allergies</label>
              <textarea
                  id="dietary"
                  v-model="bookingForm.dietaryRestrictions"
                  placeholder="e.g., Vegetarian, Gluten-free, Nut allergy..."
                  rows="2"
              ></textarea>
            </div>
            <div class="form-group">
              <label for="requests">Special Requests</label>
              <textarea
                  id="requests"
                  v-model="bookingForm.specialRequests"
                  placeholder="e.g., Wheelchair accessible, Birthday celebration..."
                  rows="2"
              ></textarea>
            </div>
          </div>

          <!-- Payment Section -->
          <div class="form-section">
            <h4>Payment</h4>
            <div class="payment-summary">
              <div class="payment-row">
                <span>{{ partySize }} × €{{ selectedDateInfo.pricePerPerson }}</span>
                <span>€{{ totalPrice }}</span>
              </div>
              <div class="payment-total">
                <span>Total Amount</span>
                <span>€{{ totalPrice }}</span>
              </div>
            </div>

            <!-- Simple Stripe Payment Info -->
            <div class="payment-element">
              <div class="stripe-payment-box">
                <div class="stripe-header">
                  <span class="credit-card-icon">💳</span>
                  <span>Secure payment with Stripe</span>
                </div>
                <div class="payment-methods-text">
                  Credit card, iDEAL, and more payment methods
                </div>
              </div>
            </div>
          </div>

          <!-- Form Actions -->
          <div class="form-actions">
            <button type="button" class="cancel-button" @click="closeBookingForm">
              Cancel
            </button>
            <button type="submit" class="confirm-booking-button" :disabled="isProcessing">
              <span v-if="!isProcessing">Complete Booking - €{{ totalPrice }}</span>
              <span v-else>Processing...</span>
            </button>
          </div>
        </form>
      </div>
    </transition>

    <!-- Loading state -->
    <div v-if="isLoading" class="loading">
      <div class="loading-spinner"></div>
      <span>Loading availability...</span>
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
      isLoading: false,
      showBookingForm: false,
      isProcessing: false,
      bookingForm: {
        firstName: '',
        lastName: '',
        email: '',
        phone: '',
        dietaryRestrictions: '',
        specialRequests: ''
      }
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
      const pricePerPerson = parseFloat(tour.tour.pricePerPerson || 0);

      return {
        tourName: tour.tour.name,
        currentBookings,
        maxCapacity,
        availableSpots,
        canAccommodateParty,
        pricePerPerson: pricePerPerson.toFixed(2)
      };
    },
    capacityPercentage() {
      if (!this.selectedDateInfo) return 0;
      return (this.selectedDateInfo.currentBookings / this.selectedDateInfo.maxCapacity) * 100;
    },
    totalPrice() {
      if (!this.selectedDateInfo) return '0.00';
      const total = parseFloat(this.selectedDateInfo.pricePerPerson) * this.partySize;
      return total.toFixed(2);
    }
  },
  methods: {
    async fetchData() {
      this.isLoading = true;
      try {
        const year = this.currentDate.getFullYear();
        const month = this.currentDate.getMonth() + 1;

        const availabilityResponse = await axios.get(`/api/tour-availability`, {
          params: { year, month, partySize: this.partySize }
        });

        this.availableDateTimes = availabilityResponse.data.map(dateStr => ({
          date: parseISO(dateStr),
          id: dateStr
        }));

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
      this.showBookingForm = false; // Reset form when selecting new date
    },

    closeBookingForm() {
      this.showBookingForm = false;
      this.resetBookingForm();
    },

    resetBookingForm() {
      this.bookingForm = {
        firstName: '',
        lastName: '',
        email: '',
        phone: '',
        dietaryRestrictions: '',
        specialRequests: ''
      };
    },

    async processBooking() {
      this.isProcessing = true;

      try {
        // Prepare booking data for backend
        const bookingData = {
          scheduledTourId: this.getScheduledTourId(),
          partySize: this.partySize,
          totalAmount: parseFloat(this.totalPrice),
          customerInfo: this.bookingForm,
          tourDate: format(this.selectedDate, 'yyyy-MM-dd')
        };

        // Call backend to create Stripe checkout session
        const response = await axios.post('/api/bookings/create-checkout-session', bookingData);

        // Redirect to Stripe Checkout
        window.location.href = response.data.checkoutUrl;

      } catch (error) {
        console.error('Booking failed:', error);

        // Better error handling
        if (error.response && error.response.data) {
          alert(`Booking failed: ${error.response.data}`);
        } else {
          alert('Booking failed. Please try again.');
        }

        this.isProcessing = false;
      }
    },

    getScheduledTourId() {
      const tour = this.scheduledTours.find(tour =>
          isSameDay(parseISO(tour.localDate), this.selectedDate)
      );
      return tour ? tour.scheduledTourId : null;
    },

    changeMonth(delta) {
      this.currentDate = addMonths(this.currentDate, delta);
      this.selectedDate = null;
      this.showBookingForm = false;
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
.calendar {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 600px;
  margin: 0 auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.party-size-selector {
  margin-bottom: 1.5rem;
  text-align: center;
  padding: 1rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  color: white;
  width: 100%;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.party-size-selector label {
  margin-right: 0.75rem;
  font-weight: 600;
  font-size: 1.1rem;
}

.party-size-selector select {
  padding: 0.75rem 1rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  background: white;
  color: #333;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  cursor: pointer;
  transition: all 0.3s ease;
}

.party-size-selector select:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(255,255,255,0.3);
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-bottom: 1.5rem;
  padding: 0 1rem;
}

.controls button {
  padding: 0.75rem 1.5rem;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.3);
}

.controls button:hover {
  background: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.4);
}

.controls h2 {
  margin: 0;
  color: #333;
  font-size: 1.5rem;
  font-weight: 700;
}

.days-header, .days-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
  gap: 2px;
}

.day-name {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
  font-weight: 600;
  color: #666;
  background: #f8f9fa;
  border-radius: 4px;
}

.day {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 45px;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.3s ease;
  font-weight: 500;
  position: relative;
}

.day:hover {
  transform: scale(1.05);
}

.day.available {
  background: linear-gradient(135deg, #a8e6cf 0%, #88d8a3 100%);
  color: #2d5a3d;
  box-shadow: 0 2px 8px rgba(168, 230, 207, 0.4);
}

.day.available:hover {
  background: linear-gradient(135deg, #88d8a3 0%, #6bb77b 100%);
  box-shadow: 0 4px 12px rgba(168, 230, 207, 0.6);
}

.day.unavailable {
  background: linear-gradient(135deg, #ffb3ba 0%, #ff9aa2 100%);
  color: #8b2635;
  cursor: not-allowed;
  box-shadow: 0 2px 8px rgba(255, 179, 186, 0.4);
}

.day.no-tour {
  background: #f5f5f5;
  color: #ccc;
  cursor: not-allowed;
}

.day.selected {
  background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
  color: white;
  font-weight: 700;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.5);
  transform: scale(1.1);
}

.capacity-info {
  margin-top: 2rem;
  padding: 0;
  border-radius: 16px;
  background: white;
  width: 100%;
  max-width: 500px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.1);
  border: 1px solid #e9ecef;
  overflow: hidden;
}

.tour-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 1.5rem;
  text-align: center;
}

.tour-header h3 {
  margin: 0 0 0.5rem 0;
  font-size: 1.4rem;
  font-weight: 700;
}

.tour-date {
  margin: 0;
  font-size: 1rem;
  opacity: 0.9;
  font-weight: 500;
}

.pricing-section {
  padding: 1.5rem;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
}

.price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
  font-size: 1rem;
}

.price-label {
  color: #666;
  font-weight: 500;
}

.price-value {
  color: #333;
  font-weight: 600;
}

.total-price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0 0.5rem 0;
  margin-top: 0.5rem;
  border-top: 2px solid #dee2e6;
  font-size: 1.2rem;
}

.total-label {
  color: #333;
  font-weight: 700;
}

.total-amount {
  color: #28a745;
  font-weight: 800;
  font-size: 1.3rem;
}

.capacity-section {
  padding: 1.5rem;
}

.capacity-header {
  margin-bottom: 1rem;
  font-weight: 600;
  color: #333;
  text-align: center;
}

.capacity-bar {
  position: relative;
  width: 100%;
  height: 35px;
  background-color: #e9ecef;
  border-radius: 18px;
  overflow: hidden;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
}

.capacity-fill {
  height: 100%;
  background: linear-gradient(90deg, #28a745 0%, #ffc107 70%, #dc3545 90%);
  transition: width 0.4s ease;
  border-radius: 18px;
}

.capacity-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: 700;
  color: #333;
  text-shadow: 1px 1px 2px rgba(255,255,255,0.8);
  font-size: 0.9rem;
}

.booking-section {
  padding: 1.5rem;
  background: #f8f9fa;
}

.booking-available {
  text-align: center;
}

.availability-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
  color: #28a745;
  font-weight: 600;
  font-size: 1.1rem;
}

.booking-unavailable {
  text-align: center;
}

.unavailability-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  color: #dc3545;
  font-weight: 600;
  font-size: 1.1rem;
  padding: 1rem;
  background: #f8d7da;
  border-radius: 8px;
  border: 1px solid #f5c6cb;
}

.book-button {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding: 1rem 1.5rem;
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: 700;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(40, 167, 69, 0.3);
}

.book-button:hover {
  background: linear-gradient(135deg, #218838 0%, #1ea085 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(40, 167, 69, 0.4);
}

.button-text {
  font-size: 1.1rem;
}

.button-price {
  font-size: 1.2rem;
  font-weight: 800;
}

.loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-top: 2rem;
  padding: 2rem;
  color: #666;
  font-style: italic;
  font-weight: 500;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid #e9ecef;
  border-top: 2px solid #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.check-icon, .cross-icon {
  font-size: 1.2rem;
}

/* Booking Form Styles */
.booking-form {
  margin-top: 1rem;
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.1);
  border: 1px solid #e9ecef;
  overflow: hidden;
  width: 100%;
  max-width: 500px;
}

.form-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.form-header h3 {
  margin: 0;
  font-size: 1.3rem;
  font-weight: 700;
}

.close-form {
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.close-form:hover {
  background: rgba(255,255,255,0.2);
}

.booking-summary-mini {
  padding: 1rem 1.5rem;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  font-size: 0.9rem;
  color: #666;
  text-align: center;
  font-weight: 500;
}

.booking-details {
  padding: 1.5rem;
}

.form-section {
  margin-bottom: 2rem;
}

.form-section h4 {
  margin: 0 0 1rem 0;
  color: #333;
  font-size: 1.1rem;
  font-weight: 600;
}

.optional {
  color: #666;
  font-weight: 400;
  font-size: 0.9rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group label {
  margin-bottom: 0.5rem;
  color: #333;
  font-weight: 500;
  font-size: 0.9rem;
}

.form-group input,
.form-group textarea {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-group textarea {
  resize: vertical;
  min-height: 60px;
}

.payment-summary {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.payment-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  color: #666;
}

.payment-total {
  display: flex;
  justify-content: space-between;
  padding-top: 0.5rem;
  border-top: 1px solid #dee2e6;
  font-weight: 700;
  color: #333;
  font-size: 1.1rem;
}

.payment-element {
  margin-bottom: 1.5rem;
}

.stripe-payment-box {
  border: 2px dashed #d1d5db;
  border-radius: 8px;
  padding: 1.5rem;
  text-align: center;
  background: #fafafa;
}

.stripe-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-weight: 600;
  font-size: 1.1rem;
  color: #374151;
  margin-bottom: 0.5rem;
}

.credit-card-icon {
  font-size: 1.2rem;
}

.payment-methods-text {
  color: #6b7280;
  font-size: 0.9rem;
  font-style: italic;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
}

.cancel-button {
  flex: 1;
  padding: 1rem;
  background: #6c757d;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.3s;
}

.cancel-button:hover {
  background: #5a6268;
}

.confirm-booking-button {
  flex: 2;
  padding: 1rem;
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 700;
  font-size: 1.1rem;
  transition: all 0.3s ease;
}

.confirm-booking-button:hover:not(:disabled) {
  background: linear-gradient(135deg, #218838 0%, #1ea085 100%);
  transform: translateY(-1px);
}

.confirm-booking-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

/* Slide down animation */
.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.4s ease;
  max-height: 1000px;
  opacity: 1;
}

.slide-down-enter-from,
.slide-down-leave-to {
  max-height: 0;
  opacity: 0;
  transform: translateY(-20px);
}

/* Responsive design */
@media (max-width: 600px) {
  .form-row {
    grid-template-columns: 1fr;
  }

  .form-actions {
    flex-direction: column;
  }
}
</style>