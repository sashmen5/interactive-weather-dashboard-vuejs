<template>
  <div id="ancestor">
    <div class="container-fluid" id="app">
      <div class="row">
        <div id="sidebar" class="col-md-3 col-sm-4 col-xs-12 sidebar">
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
          <div id="info">
            <div class="wrapper-left">
              <div id="current-weather">
                {{ currentWeather.temp }}
                <span>°C</span>
              </div>
              <div id="weather-desc">{{ currentWeather.summary }}</div>
              <div class="temp-max-min">
                <div class="max-desc">
                  <div id="max-detail">
                    <i>▲</i>
                    {{ currentWeather.todayHighLow.todayTempHigh }}
                    <span>°C</span>
                  </div>
                  <div id="max-summary">at {{ currentWeather.todayHighLow.todayTempHighTime }}</div>
                </div>
                <div class="min-desc">
                  <div id="min-detail">
                    <i>▼</i>
                    {{ currentWeather.todayHighLow.todayTempLow }}
                    <span>°C</span>
                  </div>
                  <div id="min-summary">at {{ currentWeather.todayHighLow.todayTempLowTime }}</div>
                </div>
              </div>
            </div>
            <div class="wrapper-right">
              <div class="date-time-info">
                <div id="date-desc">
                  <img src="./assets/calendar.svg" width="20" height="20">
                  {{ currentWeather.time }}
                </div>
              </div>
              <div class="location-info">
                <div id="location-desc">
                  <img
                    src="./assets/location.svg"
                    width="10.83"
                    height="15.83"
                    style="opacity: 0.9;"
                  >
                  {{ currentWeather.full_location }}
                  <div id="location-detail" class="mt-1">
                    Lat: {{ currentWeather.formatted_lat }}
                    <br>
                    Long: {{ currentWeather.formatted_long }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <dashboard-content
          class="col-md-9 col-sm-8 col-xs-12 content"
          id="dashboard-content"
          :highlights="highlights"
          :tempVar="tempVar"
        ></dashboard-content>
      </div>
    </div>
  </div>
</template>

<script>
  import Content from './components/Content.vue';

  export default {
    name: 'app',
    props: [],
    components: {
      'dashboard-content': Content
    },
    data() {
      return {
        weatherDetails: false,
        location: '', // raw location from input
        lat: '', // raw latitude from google maps api response
        long: '', // raw longitude from google maps api response
        completeWeatherApi: '', // weather api string with lat and long
        rawWeatherData: '', // raw response from weather api
        currentWeather: {
          full_location: '', // for full address
          formatted_lat: '', // for N/S
          formatted_long: '', // for E/W
          time: '',
          temp: '',
          todayHighLow: {
            todayTempHigh: '',
            todayTempHighTime: '',
            todayTempLow: '',
            todayTempLowTime: ''
          },
          summary: '',
          possibility: ''
        },
        tempVar: {
          tempToday: [
            // gets added dynamically by this.getSetHourlyTempInfoToday()
          ],
        },
        highlights: {
          uvIndex: '',
          visibility: '',
          windStatus: {
            windSpeed: '',
            windDirection: '',
            derivedWindDirection: ''
          },
        }
      };
    },
    methods: {
      // Some utility functions
      convertToTitleCase: function(str) {
        str = str.toLowerCase().split(' ');
        for (var i = 0; i < str.length; i++) {
          str[i] = str[i].charAt(0).toUpperCase() + str[i].slice(1);
        }
        return str.join(' ');
      },
      formatPossibility: function(str) {
        str = str.toLowerCase().split('-');
        for (var i = 0; i < str.length; i++) {
          str[i] = str[i].charAt(0).toUpperCase() + str[i].slice(1);
        }
        return str.join(' ');
      },
      unixToHuman: function(timezone, timestamp) {
        /* READ THIS BEFORE JUDGING & DEBUGGING
        For any location beyond the arctic circle and the
        antarctic circle, the goddamn weather api does not return certain
        keys/values in each of this.rawWeatherData.daily.data[some_array_index].
        Due to this, console throws up an error.
        The code is correct, the problem is with the API.
        May be later on I will add some padding to tackle missing values.
        */
        var moment = require('moment-timezone'); // for handling date & time
        var decipher = new Date(timestamp * 1000);
        var human = moment(decipher)
          .tz(timezone)
          .format('llll');
        var timeArray = human.split(' ');
        var timeNumeral = timeArray[4];
        var timeSuffix = timeArray[5];
        var justTime = timeNumeral + ' ' + timeSuffix;
        var monthDateArray = human.split(',');
        var monthDate = monthDateArray[1].trim();
        return {
          fullTime: human,
          onlyTime: justTime,
          onlyMonthDate: monthDate
        };
      },
      fahToCel: function(tempInFahrenheit) {
        var tempInCelcius = Math.round((5 / 9) * (tempInFahrenheit - 32));
        return tempInCelcius;
      },
      milibarToKiloPascal: function(pressureInMilibar) {
        var pressureInKPA = pressureInMilibar * 0.1;
        return Math.round(pressureInKPA);
      },
      mileToKilometer: function(miles) {
        var kilometer = miles * 1.60934;
        return Math.round(kilometer);
      },
      deriveWindDir: function(windDir) {
        var wind_directions_array = [
          { minVal: 0, maxVal: 30, direction: 'N' },
          { minVal: 31, maxVal: 45, direction: 'NNE' },
          { minVal: 46, maxVal: 75, direction: 'NE' },
          { minVal: 76, maxVal: 90, direction: 'ENE' },
          { minVal: 91, maxVal: 120, direction: 'E' },
          { minVal: 121, maxVal: 135, direction: 'ESE' },
          { minVal: 136, maxVal: 165, direction: 'SE' },
          { minVal: 166, maxVal: 180, direction: 'SSE' },
          { minVal: 181, maxVal: 210, direction: 'S' },
          { minVal: 211, maxVal: 225, direction: 'SSW' },
          { minVal: 226, maxVal: 255, direction: 'SW' },
          { minVal: 256, maxVal: 270, direction: 'WSW' },
          { minVal: 271, maxVal: 300, direction: 'W' },
          { minVal: 301, maxVal: 315, direction: 'WNW' },
          { minVal: 316, maxVal: 345, direction: 'NW' },
          { minVal: 346, maxVal: 360, direction: 'NNW' }
        ];
        var wind_direction = '';
        for (var i = 0; i < wind_directions_array.length; i++) {
          if (
            windDir >= wind_directions_array[i].minVal &&
            windDir <= wind_directions_array[i].maxVal
          ) {
            wind_direction = wind_directions_array[i].direction;
          }
        }
        return wind_direction;
      },

      // Some basic action oriented functions
      makeInputEmpty: function() {
        this.$refs.input.value = '';
      },
      makeTempVarTodayEmpty: function() {
        this.tempVar.tempToday = [];
      },
      detectEnterKeyPress: function() {
        var input = this.$refs.input;
        input.addEventListener('keyup', function(event) {
          event.preventDefault();
          var enterKeyCode = 13;
          if (event.keyCode === enterKeyCode) {
            this.setHitEnterKeyTrue();
          }
        });
      },
      locationEntered: function() {
        var input = this.$refs.input;
        if (input.value === '') {
          this.location = "New York";
        } else {
          this.location = this.convertToTitleCase(input.value);
        }
        this.makeInputEmpty();
        this.makeTempVarTodayEmpty();
      },

      getCoordinates: function() {
        this.locationEntered();
        var loc = this.location;
        var coords;
        var geocoder = new google.maps.Geocoder();
        return new Promise(function(resolve, reject) {
          geocoder.geocode({ address: loc }, function(results, status) {
            if (status == google.maps.GeocoderStatus.OK) {
              this.lat = results[0].geometry.location.lat();
              this.long = results[0].geometry.location.lng();
              this.full_location = results[0].formatted_address;
              coords = {
                lat: this.lat,
                long: this.long,
                full_location: this.full_location
              };
              resolve(coords);
            } else {
              alert("Oops! Couldn't get data for the location");
            }
          });
        });
      },

      // Some basic asynchronous functions
      setFormatCoordinates: async function() {
        var coordinates = await this.getCoordinates();
        this.lat = coordinates.lat;
        this.long = coordinates.long;
        this.currentWeather.full_location = coordinates.full_location;
        // Remember to beautify lat for N/S
        if (coordinates.lat > 0) {
          this.currentWeather.formatted_lat =
            (Math.round(coordinates.lat * 10000) / 10000).toString() + '°N';
        } else if (coordinates.lat < 0) {
          this.currentWeather.formatted_lat =
            (-1 * (Math.round(coordinates.lat * 10000) / 10000)).toString() +
            '°S';
        } else {
          this.currentWeather.formatted_lat = (
            Math.round(coordinates.lat * 10000) / 10000
          ).toString();
        }
        // Remember to beautify long for N/S
        if (coordinates.long > 0) {
          this.currentWeather.formatted_long =
            (Math.round(coordinates.long * 10000) / 10000).toString() + '°E';
        } else if (coordinates.long < 0) {
          this.currentWeather.formatted_long =
            (-1 * (Math.round(coordinates.long * 10000) / 10000)).toString() +
            '°W';
        } else {
          this.currentWeather.formatted_long = (
            Math.round(coordinates.long * 10000) / 10000
          ).toString();
        }
      },
      fixWeatherApi: async function() {
        await this.setFormatCoordinates();
        const weatherApi =
          'https://csm.fusioncharts.com/files/assets/wb/wb-data.php?src=darksky&lat=' +
          this.lat +
          '&long=' +
          this.long;
        this.completeWeatherApi = weatherApi;
      },
      fetchWeatherData: async function() {
        await this.fixWeatherApi();
        const axios = require('axios'); // for handling weather api promise
        const weatherApiResponse = await axios.get(this.completeWeatherApi);
        if (weatherApiResponse.status === 200) {
          this.rawWeatherData = weatherApiResponse.data;
        } else {
          alert('Hmm... Seems like our weather experts are busy!');
        }
      },

      // Get and set functions; often combined, because they are short

      // For basic info — left panel/sidebar
      getTimezone: function() {
        return this.rawWeatherData.timezone;
      },
      getSetCurrentTime: function() {
        var currentTime = this.rawWeatherData.currently.time;
        var timezone = this.getTimezone();
        this.currentWeather.time = this.unixToHuman(
          timezone,
          currentTime
        ).fullTime;
      },
      getSetSummary: function() {
        var currentSummary = this.convertToTitleCase(
          this.rawWeatherData.currently.summary
        );
        if (currentSummary.includes(' And')) {
          currentSummary = currentSummary.replace(' And', ',');
        }
        this.currentWeather.summary = currentSummary;
      },
      getSetPossibility: function() {
        var possible = this.formatPossibility(this.rawWeatherData.daily.icon);
        if (possible.includes(' And')) {
          possible = possible.replace(' And', ',');
        }
        this.currentWeather.possibility = possible;
      },
      getSetCurrentTemp: function() {
        var currentTemp = this.rawWeatherData.currently.temperature;
        this.currentWeather.temp = this.fahToCel(currentTemp);
      },
      getTodayDetails: function() {
        return this.rawWeatherData.daily.data[0];
      },
      getSetTodayTempHighLowWithTime: function() {
        var timezone = this.getTimezone();
        var todayDetails = this.getTodayDetails();
        this.currentWeather.todayHighLow.todayTempHigh = this.fahToCel(
          todayDetails.temperatureMax
        );
        this.currentWeather.todayHighLow.todayTempHighTime = this.unixToHuman(
          timezone,
          todayDetails.temperatureMaxTime
        ).onlyTime;
        this.currentWeather.todayHighLow.todayTempLow = this.fahToCel(
          todayDetails.temperatureMin
        );
        this.currentWeather.todayHighLow.todayTempLowTime = this.unixToHuman(
          timezone,
          todayDetails.temperatureMinTime
        ).onlyTime;
      },
      getHourlyInfoToday: function() {
        return this.rawWeatherData.hourly.data;
      },
      getSetHourlyTempInfoToday: function() {
        var unixTime = this.rawWeatherData.currently.time;
        var timezone = this.getTimezone();
        var todayMonthDate = this.unixToHuman(timezone, unixTime).onlyMonthDate;
        var hourlyData = this.getHourlyInfoToday();
        for (var i = 0; i < hourlyData.length; i++) {
          var hourlyTimeAllTypes = this.unixToHuman(timezone, hourlyData[i].time);
          var hourlyOnlyTime = hourlyTimeAllTypes.onlyTime;
          var hourlyMonthDate = hourlyTimeAllTypes.onlyMonthDate;
          if (todayMonthDate === hourlyMonthDate) {
            var hourlyObject = { hour: '', temp: '' };
            hourlyObject.hour = hourlyOnlyTime;
            hourlyObject.temp = this.fahToCel(hourlyData[i].temperature).toString();
            this.tempVar.tempToday.push(hourlyObject);
            /*
            Since we are using array.push(), we are just adding elements
            at the end of the array. Thus, the array is not getting emptied
            first when a new location is entered.
            to solve this problem, a method this.makeTempVarTodayEmpty()
            has been created, and called from this.locationEntered().
            */
          }
        }
        /*
        To cover the edge case where the local time is between 10 — 12 PM,
        and therefore there are only two elements in the array
        this.tempVar.tempToday. We need to add the points for minimum temperature
        and maximum temperature so that the chart gets generated with atleast four points.
        */
        if (this.tempVar.tempToday.length <= 2) {
          var minTempObject = {
            hour: this.currentWeather.todayHighLow.todayTempHighTime,
            temp: this.currentWeather.todayHighLow.todayTempHigh
          };
          var maxTempObject = {
            hour: this.currentWeather.todayHighLow.todayTempLowTime,
            temp: this.currentWeather.todayHighLow.todayTempLow
          };
          /*
          Typically, lowest temp are at dawn,
          highest temp is around mid day.
          Thus we can safely arrange like min, max, temp after 10 PM.
          */
          // array.unshift() adds stuff at the beginning of the array.
          // the order will be: min, max, 10 PM, 11 PM.
          this.tempVar.tempToday.unshift(maxTempObject, minTempObject);
        }
      },

      // For Today Highlights
      getSetUVIndex: function() {
        var uvIndex = this.rawWeatherData.currently.uvIndex;
        this.highlights.uvIndex = uvIndex;
      },
      getSetVisibility: function() {
        var visibilityInMiles = this.rawWeatherData.currently.visibility;
        this.highlights.visibility = this.mileToKilometer(visibilityInMiles);
      },
      getSetWindStatus: function() {
        var windSpeedInMiles = this.rawWeatherData.currently.windSpeed;
        this.highlights.windStatus.windSpeed = this.mileToKilometer(
          windSpeedInMiles
        );
        var absoluteWindDir = this.rawWeatherData.currently.windBearing;
        this.highlights.windStatus.windDirection = absoluteWindDir;
        this.highlights.windStatus.derivedWindDirection = this.deriveWindDir(
          absoluteWindDir
        );
      },

      // top level for info section
      organizeCurrentWeatherInfo: function() {
        // data in this.currentWeather
        /*
        Coordinates and location is covered (get & set) in:
        — this.getCoordinates()
        — this.setFormatCoordinates()
        There are lots of async-await involved there.
        So it's better to keep them there.
        */
        this.getSetCurrentTime();
        this.getSetCurrentTemp();
        this.getSetTodayTempHighLowWithTime();
        this.getSetSummary();
        this.getSetPossibility();
      },
      organizeTodayHighlights: function() {
        // top level for highlights
        this.getSetUVIndex();
        this.getSetVisibility();
        this.getSetWindStatus();
      },

      // topmost level orchestration
      organizeAllDetails: async function() {
        // top level organization
        await this.fetchWeatherData();
        this.organizeCurrentWeatherInfo();
        this.organizeTodayHighlights();
        this.getSetHourlyTempInfoToday();
      },
    },
    mounted: async function() {
      this.location = "New York";
      await this.organizeAllDetails();
    }
  };
</script>
