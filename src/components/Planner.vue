<template>
  <!-- ACCORDION -->
  <div class="accordion accordion-flush" id="accordionFlushExample">
    <div class="accordion-item">
      <h2 class="accordion-header">
        <button
          class="accordion-button collapsed"
          type="button"
          data-bs-toggle="collapse"
          :data-bs-target="`#flush-collapseOne-` + day"
          aria-expanded="false"
          :aria-controls="`flush-collapseOne-` + day"
        >
          GIORNO {{ day }}
        </button>
      </h2>
      <div
        :id="`flush-collapseOne-` + day"
        class="accordion-collapse collapse"
        data-bs-parent="#accordionFlushExample"
      >
        <div class="accordion-body">
          <template v-if="myTrip.activities">
            <div class="row" v-for="activity in myTrip.activities[day - 1]">
              <div class="col-4">{{ activity.name }}</div>
              <div class="col-4">{{ activity.time }}</div>
              <div class="col-4">{{ activity.note }}</div>
            </div>
          </template>

          <button type="button" @click="openModal(day)">Aggiungi</button>
        </div>
      </div>
    </div>
  </div>
  <!-- MODAL -->
  <div class="customModal" :id="'activity-modal-' + day">
    <div class="close" @click="closeModal(day)">X</div>
    <div>
      <h3>GIORNO {{ day }}</h3>
      <div class="row flex-column justify-space-around">
        <div class="col-6">
          <div>
            <p>Nome</p>
            <input type="text" placeholder="Nome" v-model="activity.name" required />
          </div>
          <div>
            <p>Luogo</p>
            <input type="text" placeholder="Location" v-model="searchTerm" required />
            <br />
            <button class="btn btn-warning" @click="search()">Localizza</button>
          </div>
          <div>
            <p>Ora</p>
            <input type="time" v-model="activity.time" />
          </div>

          <div>
            <p>Note</p>
            <textarea v-model="activity.note"></textarea>
          </div>

          <button @click="pushActivity(day)">Aggiungi</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import tt from '@tomtom-international/web-sdk-maps'
import axios from 'axios'

export default {
  data() {
    return {
      // myTrip: {},
      searchTerm: '',
      activity: {
        name: '',
        time: '',
        note: '',
        locationCoordinates: []
      }
    }
  },
  props: { day: Number, myTrip: Object },
  methods: {
    pushActivity(day) {
      let index = day - 1
      this.myTrip.activities[index].push(this.activity)
      let trips = localStorage.getItem('trips')
      const myTrips = JSON.parse(trips)
      let currentUrl = window.location.href
      let tripId = currentUrl.substring(currentUrl.lastIndexOf('/') + 1)
      myTrips[tripId] = this.myTrip
      localStorage.setItem('trips', JSON.stringify(myTrips))
      this.closeModal(day)
      // this.fetchTrips()
    },
    openModal(day) {
      let modal = document.getElementById('activity-modal-' + day)
      modal.style.display = 'block'
    },
    closeModal(day) {
      let modal = document.getElementById('activity-modal-' + day)
      modal.style.display = 'none'
    },
    async search() {
      await axios
        .get(
          `https://api.tomtom.com/search/2/search/${this.searchTerm}.json?key=qicXsxpR2GuZBNAJUJA0XBi1PHp4OzgD&limit=10&language=it-IT`
        )
        .then((response) => {
          console.log(response)

          let lat = response.data.results[0].position.lat
          let lon = response.data.results[0].position.lon
          this.activity.locationCoordinates = [lon, lat]
        })
        .catch((error) => {
          console.error(error)
        })
    }
  },

  mounted() {
    // this.fetchTrips()
  }
}
</script>

<style lang="scss" scoped>
.customModal {
  display: none;
  background-color: white;
  z-index: 5;
  width: auto;
  overflow: hidden;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 10px 5px 0 rgba(black, 0.2);

  .close {
    position: absolute;
    right: 30px;
    top: 20px;
    &:hover {
      transform: scale(1.1);
    }
  }
}
</style>
