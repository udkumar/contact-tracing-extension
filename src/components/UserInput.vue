<template>
  <!-- <div style="background-color: green !important"> -->
  <b-container style="margin-top: 15px !important">
    <b-row>
      <b-col cols="7" class="userinput-section1">
        <div class="userinput-section1-header">
          <h2>Todo Form</h2>
          <button @click="download"><img src="icons/download.png" alt="" /></button>
        </div>
        <div class="userinput-section1-inputbox">
          <input v-if="!userNameSet" type="text" id="userEnteredDate" placeholder="Full name" v-model="userName" />
          <button class="setNameButton" v-if="!userNameSet" @click="setUserName" :disabled="setNamebtnDisabled">
            Set name
          </button>
          <input type="date" v-model="userDate" id="userEnteredDate" />
          <input type="text" name="userEnteredLocation" id="autocomplete" ref="origin" />
        </div>
        <p
          :style="[showAddedLine ? { color: 'green' } : { color: 'white' }]"
          style="margin-left: 10px !important; font-size: 18px !important; margin-bottom: 0 !important"
        >
          Added!
        </p>
        <div class="userinput-section1-buttons">
          <button @click="addInput" class="addButton" :disabled="btnDisabled">Add</button>
          <button @click="ClearLocal" class="addButton">Clear local</button>
          <button @click="showResults" class="showResults">Show results</button>
          <button @click="matchResults" class="showResults">Validate Contact Tracing</button>
        </div>
        <h4 v-if="userNameSet" style="text-transform: capitalize !important;">User Name: {{ setUserName }}</h4>
        <div class="userinput-section1-matchstats" v-if="resultsMatched || pTotalIdentified">
          <p v-if="resultsMatched">Correctly identified locations out of total identified : {{ matchResult }}%</p>
          <p v-if="pTotalIdentified">Identified locations out of total available data : {{ pTotalIdentified }}%</p>
          <p v-if="userLocationValidatedByGoogle">
            Number of locations that both user and Google remember : {{ userLocationValidatedByGoogle }}
          </p>
          <p v-if="locationFoundByGoogle">
            Number of locations that Google remembers but not the user : {{ locationFoundByGoogle }}
          </p>
          <p v-if="locationGoogleNotFound">
            Number of locations that user remembers but not Google : {{ locationGoogleNotFound }}
          </p>
          <p>
            Number of locations close to 150 meter:
            {{ closeLocationsResultLoading ? 'Loading...' : closeLocationsResult }}
          </p>
        </div>
        <h3 v-if="results.length == 0" style="margin-top: 25px !important">No Data</h3>
        <table v-else style="width:100%" id="user-results-table">
          <tr>
            <!-- <th>S. No</th> -->
            <th style="width: 25% !important">Date</th>
            <th style="width: 20% !important">Name</th>
            <th style="width: 40% !important">Address</th>
            <th style="width: 15% !important">Matched?</th>
          </tr>
          <tr v-for="(result, i) in results" :key="i">
            <!-- <td>{{ i }}</td> -->
            <td style="width: 25% !important">{{ result.date }}</td>
            <td style="width: 20% !important">{{ result.name }}</td>
            <td style="width: 40% !important">{{ result.address }}</td>
            <td v-if="result.matched == '-'" style="width: 15% !important">{{ result.matched }}</td>
            <td v-if="result.matched == 'Yes'" style="width: 15% !important">
              <img style="width: 20px !important" src="icons/tick.png" />
            </td>
            <td v-if="result.matched == 'No'" style="width: 15% !important">
              <img style="width: 20px !important" src="icons/cross.png" />
            </td>
          </tr>
        </table>
      </b-col>
      <b-col cols="5" class="userinput-section2">
        <h2 v-if="showGoogleData.length > 0">Google Timeline Data</h2>
        <table style="width:100%" v-if="showGoogleData.length > 0" id="google-timeline-table">
          <tr>
            <th style="width: 30% !important">Date</th>
            <th style="width: 30% !important">Name</th>
            <th style="width: 40% !important">Address</th>
          </tr>
          <tr v-for="(data, i) in showGoogleData" :key="i">
            <td style="width: 30% !important">{{ data.timeBegin.slice(0, 10) }}</td>
            <td style="width: 30% !important">{{ data.name }}</td>
            <td style="width: 40% !important">{{ data.address }}</td>
          </tr>
        </table>
      </b-col>
    </b-row>
  </b-container>
  <!-- </div> -->
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      userName: '',
      userDate: null,
      userLocation: '',
      userPlaceName: null,
      userPlaceAddress: null,
      userPlacePId: null,
      userPlaceId: 1,
      results: [],
      showAddedLine: false,
      googleTimelineData: [],
      compareDataGoogle: [],
      showGoogleData: [],
      pTotalIdentified: 0,
      locationGoogleNotFound: 0,
      locationFoundByGoogle: 0,
      userLocationValidatedByGoogle: 0,
      resultsMatched: false,
      userSetName: null,
      userNameSet: false,
      mapsApiReturn: null,
      closeLocationsResultLoading: true,
      closeLocationsResult: null
    }
  },
  computed: {
    matchResult() {
      if (this.results.length == 0) {
        return '0.00'
      }
      let totalLength = this.results.length
      console.log(this.results)
      let matched = this.results.filter(result => result.matched == 'Yes')
      let unmatched = this.results.filter(result => result.matched == 'No')
      this.locationGoogleNotFound = unmatched.length
      return ((matched.length * 100) / totalLength).toFixed(2)
    },
    btnDisabled() {
      if (this.userDate && this.userPlaceName && this.userPlaceAddress && this.userPlacePId && this.userNameSet) {
        return false
      }
      return true
    },
    setNamebtnDisabled() {
      if (this.userName.length > 0) {
        return false
      }
      return true
    }
  },
  methods: {
    async checkCloseLocations() {
      this.closeLocationsResultLoading = true
      this.closeLocationsResult = null
      var googleAddress = []
      if (JSON.parse(localStorage.getItem('google-tdata'))) {
        googleAddress = JSON.parse(localStorage.getItem('google-tdata'))
      }

      var finalResult = 0
      for (let i = 0; i < this.results.length; i++) {
        for (let j = 0; j < googleAddress.length; j++) {
          if (
            this.results[i]['date'] == googleAddress[j]['timeBegin'].slice(0, 10) &&
            this.results[i]['matched'] == 'No' &&
            this.results[i]['address'] &&
            googleAddress[j]['address']
          ) {
            let distanceResult = await this.mapsApi(this.results[i]['address'], googleAddress[j]['address'])
            if (distanceResult[0]['elements'][0]['distance'].value < 150) {
              finalResult += 1
              break
            }
          }
        }
      }
      this.closeLocationsResultLoading = false
      this.closeLocationsResult = finalResult
    },
    async mapsApi(i, j) {
      var origin = i
      var destination = j

      var service = new google.maps.DistanceMatrixService()
      await service
        .getDistanceMatrix({
          origins: [origin],
          destinations: [destination],
          travelMode: 'WALKING'
        })
        .then(response => {
          this.mapsApiReturn = response.rows
        })
        .catch(error => {
          console.log('Error')
        })

      return this.mapsApiReturn
    },
    setUserName() {
      localStorage.setItem('user-name', this.userName)
      this.userNameSet = true
      this.setUserName = this.userName
    },
    matchResults() {
      this.googleTimelineData = JSON.parse(localStorage.getItem('google-tdata'))

      // console.log("this is the t data")
      console.log(this.googleTimelineData)
      if (this.googleTimelineData.length > 0) {
        // this.pTotalIdentified = ((this.results.length * 100) / this.googleTimelineData.length).toFixed(2)
        //this.locationFoundByGoogle = this.googleTimelineData.length
        this.showGoogleData = this.googleTimelineData
        // this.showGData()
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
      this.resultsMatched = true
      // console.log('final data')
      // console.log(this.compareDataGoogle)
      // console.log(this.results)
      this.results.forEach(result => {
        result.matched = 'No'
      })

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

      if (this.googleTimelineData.length > 0 && this.results.length > 0) {
        let initMatch = this.results.filter(result => result.matched == 'Yes')
        this.userLocationValidatedByGoogle = initMatch.length
        this.pTotalIdentified = ((initMatch.length * 100) / this.googleTimelineData.length).toFixed(2)
      } else {
        this.pTotalIdentified = 0
        this.userLocationValidatedByGoogle = 0
      }

      this.showGData()
    },
    showGData() {
      var initGData = []
      console.log(this.results)
      for (let i of this.showGoogleData) {
        var currentCompare = []
        for (let j of this.results) {
          if (i.timeBegin.slice(0, 10) == j.date) {
            if (i.name == j.name) {
              currentCompare.push(1)
              // break
            } else if (i.address && j.address) {
              let comparedResult = this.compareArray(i.address, j.address)
              if (comparedResult == 'Yes') {
                currentCompare.push(1)
              } else {
                currentCompare.push(0)
              }
            } else {
              currentCompare.push(0)
            }
          } else {
            currentCompare.push(0)
          }
        }
        if (!currentCompare.includes(1)) {
          initGData.push(i)
        }
      }

      this.showGoogleData = initGData
      this.locationFoundByGoogle = this.showGoogleData.length
      this.checkCloseLocations()

      // for (let i in this.results) {
      //   // var matched = 'Nil'
      //   for (let j of this.showGoogleData) {
      //     if (j.timeBegin.slice(0,10) == this.results[i]['date']) {
      //       if (j.name == this.results[i]['name']) {
      //         matched = 'Yes'
      //         this.results[i]['matched'] = 'Yes'
      //         break
      //       }
      //       if (j.address && this.results[i]['address']) {
      //         let comparedResult = this.compareArray(j.address, this.results[i]['address'])
      //         if (comparedResult == 'Yes') {
      //           this.results[i]['matched'] = 'Yes'
      //         }
      //       }
      //     }
      //   }
      // }
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
    download() {
      window.print()
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
      this.userSetName = ''
      this.userNameSet = ''
      this.userName = ''
      window.location.reload()
    },
    addInput() {
      // console.log(this.userDate)
      // // console.log(this.$refs.origin.value)
      // console.log(this.userPlaceName)
      // console.log(this.userPlaceAddress)
      // console.log(this.userPlacePId)
      // console.log(this.userPlaceName)

      if (this.userDate && this.userPlaceName && this.userPlaceAddress && this.userPlacePId && this.userNameSet) {
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
          matched: '-'
        }
        toAddMain.push(toAdd)

        localStorage.setItem('savedAddress', JSON.stringify(toAddMain))
        this.userPlaceId += 1
        this.showAddedLine = true

        setTimeout(() => {
          this.showAddedLine = false
        }, 1500)
        this.userDate = null
        this.userPlaceName = null
        this.userPlaceAddress = null
        this.userPlacePId = null
        // this.userName = ''
        this.$refs.origin.value = ''
        // this.userLocation = ''
        // savedAddress = JSON.parse(localStorage.getItem('savedAddress'));
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

    if (localStorage.getItem('user-name')) {
      this.userNameSet = true
      this.setUserName = localStorage.getItem('user-name')
    } else {
      this.userNameSet = false
    }
    this.resultsMatched = false
  }
}
</script>

<style scoped lang="scss">
// div > * {
//   display: block !important;
//   margin: 10px 0 !important;
// }

// .btnBox {
//   // width: 100% !important;
//   display: flex !important;
// }

// .btnBox p {
//   margin: 0 0 0 15px !important;
// }

button:disabled,
button[disabled] {
  border: 1px solid #999999 !important;
  background-color: #cccccc !important;
  color: #666666 !important;
}

.userinput-section1 {
  // background-color: lightblue !important;

  &-header {
    display: flex !important;
    align-items: center !important;
    margin-bottom: 15px !important;

    & h2 {
      margin: 0 !important;
    }

    & button {
      width: 35px !important;
      height: 35px !important;
      margin-left: 20px !important;
      border: none !important;

      & img {
        width: 100% !important;
        height: 100% !important;
        object-fit: contain !important;
      }
    }
  }

  &-inputbox {
    width: 70% !important;
    margin-bottom: 2px !important;

    & > * {
      width: 100% !important;
      margin: 10px !important;
    }
  }

  &-buttons {
    width: 70% !important;
    display: flex !important;
    flex-wrap: wrap !important;
    justify-content: space-around !important;
    // background-color: rgb(156, 26, 26) !important;
    margin-top: 12px !important;
    margin-bottom: 25px !important;

    & > * {
      width: 35% !important;
      margin-bottom: 15px !important;
      border: none;
      background: lightcoral !important;
      color: black !important;
      padding: 8px 0 !important;
      border-radius: 10px;
    }
  }

  &-matchstats {
    height: fit-content;
    // background-color: lightgreen;
    border-radius: 10px !important;
    padding: 12px 0 12px 10px !important;
    box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px !important;
    margin: 35px 0 15px 0 !important;

    & > * {
      margin: 0 0 8px 0 !important;
    }
  }
}

input {
  // border-radius: 10px;
  height: 44px !important;
  padding-left: 8px;
  // border: 1px solid black;
}

.setNameButton {
  height: fit-content !important;
  padding: 10px !important;
  border: none;
  background: lightcoral !important;
  color: black !important;
}

#user-results-table {
  margin-top: 30px !important;
}

#google-timeline-table {
  margin-top: 16px !important;
}

#user-results-table tr:nth-child(even),
#google-timeline-table tr:nth-child(even) {
  background-color: #f2f2f2 !important;
}

#user-results-table th,
#google-timeline-table th {
  padding: 8px 0 !important;
}

#user-results-table td,
#google-timeline-table td {
  padding: 16px 18px 16px 0 !important;
}

#user-results-table > *:first-child,
#google-timeline-table > *:first-child {
  background-color: #5f3737 !important;
  color: antiquewhite !important;
}

#user-results-table tr > *:first-child,
#google-timeline-table tr > *:first-child {
  padding-left: 8px !important;
}

#google-timeline-table td {
  color: red !important;
}

button,
button:focus {
  outline: none;
}

.userinput-section2 {
  // background-color: rgb(186, 230, 173) !important;
}
</style>
