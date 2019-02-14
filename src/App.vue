<template>
  <div id="app">
    <div id="search">
      <input
        id="location-input"
        type="text"
        ref="input"
        placeholder="Location?"
        @keyup.enter="organizeAllDetails"
      >
      <button id="search-btn" @click="organizeAllDetails">
        <img src="./assets/Search.svg" width="24" height="24">
      </button>
    </div>

    <dashboard-content :highlights="highlights" :tempVar="tempVar"></dashboard-content>
  </div>
</template>

<script>
  import Content from './components/Content.vue'

  export default {
    name: 'app',
    components: {
      'dashboard-content': Content
    },
    data () {
      return {
        tempVar: {
          tempToday: [
            {hour: '11.00 AM', temp: '35'},
            {hour: '12.00 PM', temp: '36'},
            {hour: '1.00 PM', temp: '37'},
            {hour: '2.00 PM', temp: '38'},
            {hour: '3.00 PM', temp: '36'},
            {hour: '4.00 PM', temp: '35'},
          ],
        },
        highlights: {
          uvIndex: 4,
          visibility: 10,
          windStatus: {
            windSpeed: '30 km/h',
            windDirection: '30',
            derivedWindDirection: 'NNE',
          },
        },
      }
    },
    methods: {
      organizeAllDetails() {
        console.log("123")
      }
    },
    computed: {

    },
  }
</script>

<style>

</style>
