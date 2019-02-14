<template>
  <div>
    <uv-index :highlights="highlights"></uv-index>
    <visibility :highlights="highlights"></visibility>
    <wind-status :highlights="highlights"></wind-status>
  </div>
</template>

<script>
  import UVIndex from './UVIndex.vue';
  import Visibility from './Visibility.vue';
  import WindStatus from './WindStatus.vue';

  export default {
    props: ["highlights"],
    components: {
      'uv-index': UVIndex,
      'visibility': Visibility,
      'wind-status': WindStatus,
    },
    data () {
      return {

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
