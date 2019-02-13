<template>
  <div id="windstatus">
    <p>Wind Status:</p>
    <p>Speed — {{ speed }}; Direction — {{ direction }}</p>
  </div>
</template>

<script>

  export default {
    props: ["highlights"],
    data () {
      return {
        speed: this.highlights.windstatus.speed,
        direction: this.highlights.windstatus.direction
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
