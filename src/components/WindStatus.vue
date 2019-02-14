<template>
  <div>
    <p>Wind Speed — {{ windSpeed }}</p>
    <p>Wind Direction — {{ derivedWindDirection }}, or {{ windDirection }} degree clockwise with respect to true N as 0 degree.</p>
  </div>
</template>

<script>

  export default {
    props: ["highlights"],
    data () {
      return {
        windSpeed: this.highlights.windStatus.windSpeed,
        derivedWindDirection: this.highlights.windStatus.derivedWindDirection,
        windDirection: this.highlights.windStatus.windDirection
      }
    },
    methods: {

    },
    computed: {

    },
  }
</script>

<style>

</style>
