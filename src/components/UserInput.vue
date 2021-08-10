<template>
  <div>
    <input type="date" name="userEnteredDate" id="userEnteredDate" />
    <input type="text" name="userEnteredLocation" id="userEnteredLocation" v-model="userLocation" />
    <button @click="addInput" class="addButton">Add</button>
    hey77kll90
    <ul>
      <li v-for="(result, i) in searchResults" :key="i">
        {{ result }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userDate: null,
      userLocation: '',
      searchResults: [],
      service: null
    }
  },
  methods: {
    addInput() {
      console.log(window)
    },
    MapsInit() {
      this.service = new window.google.maps.places.AutocompleteService()
    },
    displaySuggestions(predictions, status) {
      if (status !== window.google.maps.places.PlacesServiceStatus.OK) {
        this.searchResults = []
        return
      }
      this.searchResults = predictions.map(prediction => prediction.description)
    }
  },
  metaInfo() {
    return {
      script: [
        {
          src: `https://maps.googleapis.com/maps/api/js?key=AIzaSyCTkRle9nCGmlaJZ7cKkGJ6xfTwSOOgiKo&libraries=places`,
          async: true,
          defer: true,
          callback: () => this.MapsInit()
        }
      ]
    }
  },
  watch: {
    userLocation(newValue) {
      if (newValue) {
        this.service.getPlacePredictions(
          {
            input: this.userLocation,
            types: ['(cities)']
          },
          this.displaySuggestions
        )
      }
    }
  },
  mounted() {
    // const autocomplete = new google.maps.places.Autocomplete(this.$refs['origin'])
  }
}
</script>

<style scoped></style>
