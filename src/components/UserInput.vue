<template>
  <div>
    <input type="date" v-model="userDate" id="userEnteredDate" />
    <input type="text" name="userEnteredLocation" id="autocomplete" v-model="userLocation" ref="origin" />
    <div class="btnBox">
      <button @click="addInput" style="margin: 0 !important" class="addButton">Add</button>
      <p v-if="showAddedLine" style="color: green !important;">Added!</p>
    </div>
    <button @click="ClearLocal" class="addButton">Clear local</button>
    <button @click="showResults" class="showResults">Show results</button>
    <button @click="matchResults" class="showResults">Match results</button>

    heypp
    <p>Match Result : {{ matchResult }}%</p>
    <table style="width:100%">
      <tr>
        <th>S. No</th>
        <th>Date</th>
        <th>Name</th>
        <th>address</th>
        <th>Matched?</th>
      </tr>
      <tr v-if="results.length == 0">
        <td>no data</td>
      </tr>
      <tr v-else v-for="(result, i) in results" :key="i">
        <td>{{ i }}</td>
        <td>{{ result.date }}</td>
        <td>{{ result.name }}</td>
        <td>{{ result.address }}</td>
        <td>{{ result.matched }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userDate: null,
      userLocation: '',
      userPlaceName: null,
      userPlaceAddress: null,
      userPlacePId: null,
      userPlaceId: 1,
      results: [],
      showAddedLine: false,
      googleTimelineData: [],
      compareDataGoogle: []
    }
  },
  computed: {
    matchResult() {
      let totalLength = this.results.length
      let matched = this.results.filter(result => result.matched == 'Yes')
      return ((matched.length * 100) / totalLength).toFixed(2)
    }
  },
  methods: {
    matchResults() {
      this.googleTimelineData = JSON.parse(localStorage.getItem('google-tdata'))
      // console.log("this is the t data")
      // console.log(this.googleTimelineData)
      if (this.googleTimelineData.length > 0) {
        for (let i of this.googleTimelineData) {
          if (i.address) {
            this.compareDataGoogle.push({
              date: i.timeBegin.slice(0, 10),
              name: i.name,
              address: i.address
            })
          } else {
            this.compareDataGoogle.push({
              date: i.timeBegin.slice(0, 10),
              name: i.name
            })
          }
        }
      }
      // console.log('final data')
      // console.log(this.compareDataGoogle)
      // console.log(this.results)
      for (let i in this.results) {
        var matched = 'Nil'
        for (let j of this.compareDataGoogle) {
          if (j.date == this.results[i]['date']) {
            if (j.name == this.results[i]['name']) {
              matched = 'Yes'
              this.results[i]['matched'] = 'Yes'
              break
            }
            if (j.address && this.results[i]['address']) {
              let comparedResult = this.compareArray(j.address, this.results[i]['address'])
              if (comparedResult == 'Yes') {
                this.results[i]['matched'] = 'Yes'
              }
            }
          }
        }
      }
    },
    compareArray(googleAddress, userAddress) {
      // console.log("compareddddddddddddd")
      // console.log(googleAddress, 'USERRRRRRRRRRRRRRRRRR', userAddress)
      var googleAArray = googleAddress.split(' ')
      var userAArray = userAddress.split(' ')
      var totalLength = googleAArray.length
      var matchLength = 0

      // console.log(googleAArray, userAArray, totalLength, matchLength)

      for (let i of userAArray) {
        if (googleAArray.includes(i)) {
          matchLength++
          continue
        }
      }
      // console.log("after loooooooop", matchLength)
      var matchLengthP = matchLength * 100

      if (matchLengthP / totalLength >= 60) {
        return 'Yes'
      } else {
        return 'Nil'
      }
    },
    showResults() {
      this.results = []
      if (JSON.parse(localStorage.getItem('savedAddress'))) {
        this.results = JSON.parse(localStorage.getItem('savedAddress'))
      }
    },
    ClearLocal() {
      localStorage.clear()
      this.userPlaceId = 1
      this.results = []
    },
    addInput() {
      // console.log(this.userDate)
      // // console.log(this.$refs.origin.value)
      // console.log(this.userPlaceName)
      // console.log(this.userPlaceAddress)
      // console.log(this.userPlacePId)
      // console.log(this.userPlaceName)

      if (this.userDate && this.userPlaceName && this.userPlaceAddress && this.userPlacePId) {
        // console.log('ok ready to add')

        // console.log(JSON.parse(localStorage.getItem('savedAddress')))

        if (JSON.parse(localStorage.getItem('savedAddress'))) {
          var toAddMain = [...JSON.parse(localStorage.getItem('savedAddress'))]
        } else {
          // alert('l')
          var toAddMain = []
        }
        // let toAddMain = [...JSON.parse(localStorage.getItem('savedAddress'))]
        // let toAddMain = []

        let toAdd = {
          id: this.userPlaceId,
          date: this.userDate,
          name: this.userPlaceName,
          address: this.userPlaceAddress,
          gid: this.userPlacePId,
          matched: 'Nil'
        }
        toAddMain.push(toAdd)

        localStorage.setItem('savedAddress', JSON.stringify(toAddMain))
        this.userPlaceId += 1
        this.showAddedLine = true
        setTimeout(() => {
          this.showAddedLine = false
        }, 1500)
        // savedAddress = JSON.parse(localStorage.getItem('savedAddress'));
      } else {
        console.log('check it ...')
      }
    }
  },
  mounted() {
    if (JSON.parse(localStorage.getItem('savedAddress'))) {
      this.results = JSON.parse(localStorage.getItem('savedAddress'))
    }

    let autocomplete = new google.maps.places.Autocomplete(document.getElementById('autocomplete'))
    autocomplete.addListener('place_changed', () => {
      const place = autocomplete.getPlace()
      console.log(place)
      this.userPlaceName = place.name
      this.userPlaceAddress = place.formatted_address
      this.userPlacePId = place.place_id
    })
  }
}
</script>

<style scoped>
div > * {
  display: block !important;
  margin: 10px 0 !important;
}

.btnBox {
  width: 100% !important;
  display: flex !important;
}

.btnBox p {
  margin: 0 0 0 15px !important;
}
</style>
