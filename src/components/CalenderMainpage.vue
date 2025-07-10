<template>
  <div class="main-container">
    <!-- Photos around the calendar -->
    <div class="photo-grid">
      <!-- Top photos -->
      <div class="photo-row top-row">
        <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[0] }" data-direction="top-left">
          <img src="../../public/photos/broodje-rookworst.jpg" alt="Food Tour 1" class="tour-photo"/>
          <div class="photo-caption">Broodje Rookworst</div>
        </div>
        <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[1] }" data-direction="top-center">
          <img src="../../public/photos/fresh-herring.jpg" alt="Food Tour 2" class="tour-photo"/>
          <div class="photo-caption">Fresh herring</div>
        </div>
        <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[2] }" data-direction="top-right">
          <img src="../../public/photos/fun.jpg" alt="Food Tour 3" class="tour-photo"/>
          <div class="photo-caption">Fun!</div>
        </div>
      </div>

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
          'past-date': isPastDate(day.date),
          'available': !isPastDate(day.date) && isAvailable(day.date),
          'unavailable': !isPastDate(day.date) && !isAvailable(day.date) && isScheduledDate(day.date),
          'no-tour': !isPastDate(day.date) && !isScheduledDate(day.date),
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
            <p class="tour-date">{{ format(selectedDate, 'MMMM d, yyyy') }} at {{ selectedDateInfo.startTime }}</p>
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
              <span class="capacity-text">{{ selectedDateInfo.currentBookings }}/{{
                  selectedDateInfo.maxCapacity
                }} people</span>
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
          <span>{{ selectedDateInfo.tourName }} • {{
              format(selectedDate, 'MMM d, yyyy')
            }} at {{ selectedDateInfo.startTime }} • {{ partySize }} people • €{{ totalPrice }}</span>
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

        <!-- Bottom photos -->
        <div class="photo-row bottom-row">
          <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[3] }" data-direction="bottom-left">
            <img src="../../public/photos/gouda-cheese.jpg" alt="Food Tour 4" class="tour-photo"/>
            <div class="photo-caption">Gouda cheese</div>
          </div>
          <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[4] }" data-direction="bottom-center">
            <img src="../../public/photos/muisjes.jpg" alt="Food Tour 5" class="tour-photo"/>
            <div class="photo-caption">Dutch 'Muisjes'</div>
          </div>
          <div class="tour-photo-container" :class="{ 'animate-in': showPhotos[5] }" data-direction="bottom-right">
            <img src="../../public/photos/streetfood.jpg" alt="Food Tour 6" class="tour-photo"/>
            <div class="photo-caption">Streetfood</div>
          </div>
        </div>
      </div>
    </div>

    <!-- Booking Success Popup -->
    <div v-if="showSuccessPopup" class="booking-success-overlay">
      <div class="booking-success-popup">
        <!-- Header -->
        <div class="popup-header">
          <button @click="closeSuccessPopup" class="close-popup">×</button>

          <div class="success-content">
            <div class="success-icon">
              <span class="check-circle">✅</span>
            </div>
            <h2>Booking Confirmed!</h2>
            <p>Your food tour has been successfully booked</p>
          </div>
        </div>

        <!-- Content -->
        <div class="popup-content">
          <!-- Booking Details -->
          <div class="booking-details-popup">
            <div class="detail-row">
              <div class="detail-icon">📅</div>
              <div>
                <p class="detail-label">Date & Time</p>
                <p class="detail-value">{{ formatPopupDate(bookingDetails?.scheduledTour?.localDate) }} at
                  {{ bookingDetails?.scheduledTour?.startTime || '12:00' }}</p>
              </div>
            </div>

            <div class="detail-row">
              <div class="detail-icon">👥</div>
              <div>
                <p class="detail-label">Party Size</p>
                <p class="detail-value">{{ bookingDetails?.numberOfPeople }}
                  {{ bookingDetails?.numberOfPeople === 1 ? 'person' : 'people' }}</p>
              </div>
            </div>

            <div class="detail-row">
              <div class="detail-icon">📍</div>
              <div>
                <p class="detail-label">Tour</p>
                <p class="detail-value">{{ bookingDetails?.scheduledTour?.tour?.name || 'Amsterdam Food Tour' }}</p>
              </div>
            </div>
          </div>

          <!-- Booking Reference -->
          <div class="booking-reference">
            <p class="reference-label">Booking Reference</p>
            <p class="reference-value">#{{ bookingRef }}</p>
          </div>

          <!-- Next Steps -->
          <div class="next-steps">
            <h3>What's Next?</h3>
            <ul>
              <li>You'll receive a confirmation email shortly</li>
              <li>Meet at the designated starting point 10 minutes early</li>
              <li>Bring a good appetite and comfortable walking shoes!</li>
            </ul>
          </div>
        </div>

        <!-- Footer -->
        <div class="popup-footer">
          <button @click="closeSuccessPopup" class="close-button">Close</button>
          <!--          <button @click="handleViewDetails" class="details-button">View Details</button>-->
        </div>
      </div>
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
  parseISO,
  isBefore,
  startOfDay
} from 'date-fns';
import axios from 'axios';

export default {
  name: 'CalendarMainpage',
  data() {
    return {
      currentDate: new Date(),
      partySize: 2,
      weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      showPhotos: [false, false, false, false, false, false],
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
      },
      // Popup related data
      showSuccessPopup: false,
      bookingDetails: null
    };
  },
  mounted() {
// Staggered bizarre animations - each photo comes in at different times
    const delays = [100, 200, 350, 450, 600, 650];
    delays.forEach((delay, index) => {
          setTimeout(() => {
            this.showPhotos[index] = true;
          }, delay);
        }
    );
  },
  computed: {
    formattedMonth() {
      return format(this.currentDate, 'MMMM yyyy');
    },
    days() {
      const startDay = startOfWeek(startOfMonth(this.currentDate));
      const endDay = endOfWeek(endOfMonth(this.currentDate));
      return eachDayOfInterval({start: startDay, end: endDay})
          .map(date => ({
            date,
            dayOfMonth: date.getDate()
          }));
    },
// Update the selectedDateInfo computed property in CalenderMainpage.vue

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
        startTime: tour.startTime || '12:00',
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
    },
    bookingRef() {
      return this.bookingDetails?.id?.toString().padStart(6, '0') || 'FT001234'
    }
  },
  methods: {
    async fetchData() {
      this.isLoading = true;
      try {
        const year = this.currentDate.getFullYear();
        const month = this.currentDate.getMonth() + 1;

        const availabilityResponse = await axios.get(`/api/tour-availability`, {
          params: {year, month, partySize: this.partySize}
        });

        this.availableDateTimes = availabilityResponse.data.map(dateStr => ({
          date: parseISO(dateStr),
          id: dateStr
        }));

        const toursResponse = await axios.get('/api/scheduled-tours', {
          params: {year, month}
        });

        this.scheduledTours = toursResponse.data;

      } catch (error) {
        console.error('Error fetching data:', error);
      } finally {
        this.isLoading = false;
      }
    },

    selectDate(date) {
      // Don't allow selection of past dates
      if (this.isPastDate(date)) return;
      if (!this.isScheduledDate(date)) return;
      this.selectedDate = date;
      this.showBookingForm = false; // Reset form when selecting new date
    },

    isPastDate(date) {
      const today = startOfDay(new Date());
      return isBefore(startOfDay(date), today);
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

    // Popup related methods
    async checkForPaymentSuccess() {
      const urlParams = new URLSearchParams(window.location.search);
      const sessionId = urlParams.get('session_id');

      if (sessionId) {
        try {
          const response = await axios.get(`/api/bookings/verify-payment`, {
            params: {session_id: sessionId}
          });

          if (response.status === 200) {
            this.bookingDetails = response.data;
            this.showSuccessPopup = true;
            // Refresh the calendar data to show updated availability
            await this.fetchData();
          }
        } catch (error) {
          console.error('Payment verification failed:', error);
        } finally {
          // Clean up URL
          window.history.replaceState({}, document.title, window.location.pathname);
        }
      }
    },

    closeSuccessPopup() {
      this.showSuccessPopup = false;
    },

    handleViewDetails() {
      console.log('View booking details:', this.bookingDetails);
      // You can add navigation to a booking details page here
      // this.$router.push(`/bookings/${this.bookingDetails.id}`);
    },

    formatPopupDate(dateString) {
      if (!dateString) return 'Date TBD'
      return new Date(dateString).toLocaleDateString('en-US', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    },

    isSameDay,
    format
  },
  async created() {
    await this.checkForPaymentSuccess();
    this.fetchData();
  }
};
</script>

<style scoped>
/* Global box-sizing fix */
* {
  box-sizing: border-box;
}

.main-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
}

.photo-grid {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  perspective: 2000px; /* Deep 3D perspective */
}

.photo-row {
  display: flex;
  justify-content: center;
  gap: 1rem;
}

.calendar-section {
  display: flex;
  justify-content: center;
  margin: 2rem 0;
}

.tour-photo-container {
  width: 180px;
  height: 140px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  position: relative;

  /* EXTREME initial state - way off screen with MASSIVE rotation */
  opacity: 0;
  transition: all 2s cubic-bezier(0.68, -0.55, 0.265, 1.55); /* Bouncy easing */
}

/* Different crazy entry animations for each position */
.tour-photo-container[data-direction="top-left"] {
  transform: translateX(-800px) translateY(-600px) rotateX(720deg) rotateY(1080deg) rotateZ(540deg) scale(0.1);
}

.tour-photo-container[data-direction="top-center"] {
  transform: translateX(0px) translateY(-800px) rotateX(-900deg) rotateY(720deg) rotateZ(-720deg) scale(0.1);
}

.tour-photo-container[data-direction="top-right"] {
  transform: translateX(800px) translateY(-600px) rotateX(1080deg) rotateY(-540deg) rotateZ(900deg) scale(0.1);
}

.tour-photo-container[data-direction="bottom-left"] {
  transform: translateX(-900px) translateY(700px) rotateX(-720deg) rotateY(1440deg) rotateZ(-1080deg) scale(0.1);
}

.tour-photo-container[data-direction="bottom-center"] {
  transform: translateX(0px) translateY(900px) rotateX(1440deg) rotateY(-900deg) rotateZ(720deg) scale(0.1);
}

.tour-photo-container[data-direction="bottom-right"] {
  transform: translateX(900px) translateY(700px) rotateX(-1080deg) rotateY(1080deg) rotateZ(-1440deg) scale(0.1);
}

/* FINAL STATE - all photos settle into place */
.tour-photo-container.animate-in {
  transform: translateX(0) translateY(0) rotateX(0deg) rotateY(0deg) rotateZ(0deg) scale(1);
  opacity: 1;
}

/* Add a subtle hover effect for fun */
.tour-photo-container:hover {
  transform: translateX(0) translateY(0) rotateX(0deg) rotateY(0deg) rotateZ(5deg) scale(1.05);
  transition: all 0.3s ease;
}

.tour-photo {
  width: 100%;
  height: 100px;
  object-fit: cover;
  display: block;
}

.photo-caption {
  padding: 0.5rem;
  text-align: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-size: 0.8rem;
  font-weight: 600;
}

/* Calendar wrapper */
.calendar {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 600px;
  margin: 0 auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Mobile responsive - stack photos vertically */
@media (max-width: 768px) {
  .photo-row {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  .tour-photo-container {
    width: 200px;
    height: 150px;
  }

  /* Reduce crazy rotations on mobile for performance */
  .tour-photo-container[data-direction*="top"] {
    transform: translateY(-400px) rotateZ(360deg) scale(0.2);
  }

  .tour-photo-container[data-direction*="bottom"] {
    transform: translateY(400px) rotateZ(-360deg) scale(0.2);
  }
}

/* Add some sparkle effects during animation */
@keyframes sparkle {
  0%, 100% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
}

.tour-photo-container.animate-in::before {
  content: '✨';
  position: absolute;
  top: -10px;
  right: -10px;
  z-index: 10;
  animation: sparkle 0.5s ease-in-out;
}

.bottom-row {
  margin-top: 2rem; /* Extra space above bottom photos */
}

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
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
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
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.3s ease;
}

.party-size-selector select:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.3);
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

.day.past-date {
  background: #f5f5f5;
  color: #ccc;
  cursor: not-allowed;
}

.day.past-date:hover {
  transform: none;
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
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
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
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
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
  text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
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
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.check-icon, .cross-icon {
  font-size: 1.2rem;
}

/* Booking Form Styles */
.booking-form {
  margin-top: 1rem;
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
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
  background: rgba(255, 255, 255, 0.2);
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

/* Form row layout - Firefox fix */
.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
  min-width: 0; /* Firefox: allows grid items to shrink */
}

/* Form groups - Firefox fix */
.form-group {
  display: flex;
  flex-direction: column;
  min-width: 0; /* Firefox: allows flex items to shrink below content size */
}

.form-group label {
  margin-bottom: 0.5rem;
  color: #333;
  font-weight: 500;
  font-size: 0.9rem;
}

/* All form inputs - Firefox fix */
.form-group input,
.form-group textarea {
  box-sizing: border-box; /* Critical for Firefox */
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  font-family: inherit; /* Use parent font */
  transition: border-color 0.3s, box-shadow 0.3s;

  /* Firefox-specific fixes */
  -moz-appearance: none; /* Remove Firefox default styling */
  outline: none; /* Remove focus outline, we'll add custom */
}

/* Placeholder styles */
.form-group input::placeholder,
.form-group textarea::placeholder {
  font-size: 0.75rem; /* Smaller than the input text (1rem) */
  color: #999; /* Lighter color */
}

/* Focus states */
.form-group input:focus,
.form-group textarea:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

/* Textarea specific */
.form-group textarea {
  resize: vertical;
  min-height: 60px;
  font-family: inherit; /* Inherit font from parent */
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

/* Booking Success Popup Styles */
.booking-success-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.booking-success-popup {
  background: white;
  border-radius: 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 450px;
  width: 100%;
  overflow: hidden;
  animation: popupAppear 0.3s ease-out;
}

@keyframes popupAppear {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.popup-header {
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
  color: white;
  padding: 2rem 1.5rem;
  text-align: center;
  position: relative;
}

.close-popup {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.close-popup:hover {
  background: rgba(255, 255, 255, 0.2);
}

.success-icon {
  margin-bottom: 1rem;
}

.check-circle {
  font-size: 3rem;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
}

.popup-header h2 {
  margin: 0 0 0.5rem 0;
  font-size: 1.5rem;
  font-weight: 700;
}

.popup-header p {
  margin: 0;
  opacity: 0.9;
  font-weight: 500;
}

.popup-content {
  padding: 1.5rem;
}

.booking-details-popup {
  margin-bottom: 1.5rem;
}

.detail-row {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.detail-icon {
  width: 40px;
  height: 40px;
  background: #f8f9fa;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
}

.detail-label {
  margin: 0;
  font-size: 0.85rem;
  color: #666;
  font-weight: 500;
}

.detail-value {
  margin: 0;
  font-weight: 600;
  color: #333;
}

.booking-reference {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1.5rem;
  text-align: center;
}

.reference-label {
  margin: 0 0 0.25rem 0;
  font-size: 0.85rem;
  color: #666;
  font-weight: 500;
}

.reference-value {
  margin: 0;
  font-family: 'Courier New', monospace;
  font-size: 1.1rem;
  font-weight: 700;
  color: #333;
}

.next-steps {
  border-top: 1px solid #e9ecef;
  padding-top: 1.5rem;
}

.next-steps h3 {
  margin: 0 0 1rem 0;
  font-size: 1rem;
  font-weight: 600;
  color: #333;
}

.next-steps ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

.next-steps li {
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  color: #666;
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
}

.next-steps li::before {
  content: '•';
  color: #28a745;
  font-weight: bold;
  margin-top: 0.1rem;
}

.popup-footer {
  padding: 1rem 1.5rem;
  background: #f8f9fa;
  border-top: 1px solid #e9ecef;
  display: flex;
  gap: 0.75rem;
}

.close-button {
  padding: 0.75rem 2rem;
  background: #6c757d;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.3s;
}

.close-button:hover {
  background: #5a6268;
}

/* Mobile-friendly layout with FULL CRAZY animations intact */

/* Keep all the extreme animations for mobile too! */
@media (max-width: 768px) {
  .main-container {
    padding: 1rem;
  }

  .photo-row {
    flex-direction: column; /* Stack photos vertically */
    align-items: center;
    gap: 1.5rem;
  }

  .tour-photo-container {
    width: 220px; /* Good mobile size */
    height: 160px;

    /* KEEP THE CRAZY ANIMATIONS - same transition */
    transition: all 2s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  }

  .tour-photo {
    width: 100%;
    height: 110px;
    object-fit: cover;
  }

  .photo-caption {
    padding: 0.75rem;
    font-size: 0.9rem;
  }

  /* KEEP ALL THE EXTREME ROTATIONS - no changes! */
  .tour-photo-container[data-direction="top-left"] {
    transform:
        translateX(-800px)
        translateY(-600px)
        rotateX(720deg)
        rotateY(1080deg)
        rotateZ(540deg)
        scale(0.1);
  }

  .tour-photo-container[data-direction="top-center"] {
    transform:
        translateX(0px)
        translateY(-800px)
        rotateX(-900deg)
        rotateY(720deg)
        rotateZ(-720deg)
        scale(0.1);
  }

  .tour-photo-container[data-direction="top-right"] {
    transform:
        translateX(800px)
        translateY(-600px)
        rotateX(1080deg)
        rotateY(-540deg)
        rotateZ(900deg)
        scale(0.1);
  }

  .tour-photo-container[data-direction="bottom-left"] {
    transform:
        translateX(-900px)
        translateY(700px)
        rotateX(-720deg)
        rotateY(1440deg)
        rotateZ(-1080deg)
        scale(0.1);
  }

  .tour-photo-container[data-direction="bottom-center"] {
    transform:
        translateX(0px)
        translateY(900px)
        rotateX(1440deg)
        rotateY(-900deg)
        rotateZ(720deg)
        scale(0.1);
  }

  .tour-photo-container[data-direction="bottom-right"] {
    transform:
        translateX(900px)
        translateY(700px)
        rotateX(-1080deg)
        rotateY(1080deg)
        rotateZ(-1440deg)
        scale(0.1);
  }

  /* Same final state */
  .tour-photo-container.animate-in {
    transform:
        translateX(0)
        translateY(0)
        rotateX(0deg)
        rotateY(0deg)
        rotateZ(0deg)
        scale(1);
    opacity: 1;
  }

  /* Keep the hover effect too */
  .tour-photo-container:hover {
    transform:
        translateX(0)
        translateY(0)
        rotateX(0deg)
        rotateY(0deg)
        rotateZ(5deg)
        scale(1.05);
    transition: all 0.3s ease;
  }

  /* Spacing adjustments only */
  .photo-grid {
    gap: 2rem;
  }

  .calendar-section {
    margin: 2rem 0;
  }
}

/* Mobile responsive */
@media (max-width: 600px) {
  .form-row {
    grid-template-columns: 1fr; /* Stack on mobile */
  }

  .form-actions {
    flex-direction: column;
  }

  .popup-footer {
    flex-direction: column;
  }
}
</style>