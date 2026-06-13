<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/appointments" class="btn btn-outline-primary">
          ⇆ Switch Page
        </router-link>
      </div>
    </nav>

    <div class="d-flex justify-content-center align-items-center" style="min-height: 80vh;">
      <div class="card shadow p-5" style="width: 100%; max-width: 700px;">
        <h2 class="text-center mb-4 text-primary">Book an Appointment</h2>

        <form class="row g-3" @submit.prevent="submitAppointment">
          <div class="col-12">
            <input
              v-model="name"
              type="text"
              class="form-control"
              placeholder="Your Name"
              required
            />
          </div>

          <div class="col-12">
            <input
              v-model="symptoms"
              type="text"
              class="form-control"
              placeholder="Symptoms"
              required
            />
          </div>

          <div class="col-12">
            <select v-model="selectedSlot" class="form-select" required>
              <option disabled value="">Select a Time Slot</option>
              <option v-for="slot in slots" :key="slot" :value="slot">
                {{ slot }}
              </option>
            </select>
          </div>

          <div class="col-12">
            <button type="submit" class="btn btn-primary w-100">
              Book
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
const API_BASE_URL = "https://my60hlfa5k.execute-api.eu-north-1.amazonaws.com";

export default {
  name: "BookAppointment",

  data() {
    return {
      name: "",
      symptoms: "",
      selectedSlot: "",
      slots: []
    };
  },

  mounted() {
    this.fetchSlots();
  },

  methods: {
    parseApiResponse(data) {
      // This handles both possible API response formats:
      // 1. Direct array response
      // 2. Lambda proxy response with body as JSON string
      if (Array.isArray(data)) {
        return data;
      }

      if (data && data.body) {
        return JSON.parse(data.body);
      }

      return [];
    },

    fetchSlots() {
      fetch(`${API_BASE_URL}/slots`)
        .then(res => {
          if (!res.ok) {
            throw new Error(`HTTP error: ${res.status}`);
          }
          return res.json();
        })
        .then(data => {
          const slotsData = this.parseApiResponse(data);

          this.slots = slotsData
            .filter(slot => slot.isBooked === false)
            .map(slot => slot.slot);
        })
        .catch(err => {
          console.error("Error loading slots:", err);
          alert("Failed to load available slots.");
        });
    },

    submitAppointment() {
      const payload = {
        patientName: this.name,
        symptoms: this.symptoms,
        slot: this.selectedSlot
      };

      fetch(`${API_BASE_URL}/appointments`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(payload)
      })
        .then(async res => {
          const data = await res.json();

          if (!res.ok) {
            throw new Error(data.message || `HTTP error: ${res.status}`);
          }

          return data;
        })
        .then(data => {
          if (data.body) {
            const parsedBody = JSON.parse(data.body);

            if (parsedBody.message === "Slot is already booked") {
              alert("This slot is already booked. Please choose another slot.");
              return;
            }
          }

          alert("Appointment booked!");

          this.name = "";
          this.symptoms = "";
          this.selectedSlot = "";

          // Refresh available slots after booking
          this.fetchSlots();
        })
        .catch(err => {
          console.error("Error booking appointment:", err);
          alert("Failed to book appointment. " + err.message);
        });
    }
  }
};
</script>
