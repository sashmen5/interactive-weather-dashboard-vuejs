<template>
  <div id="visibility">
    <p>Visibility: {{ visibility }}</p>
  </div>
</template>

<script>

  export default {
    props: ["highlights"],
    data () {
      return {
        visibility: this.highlights.visibility,
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
