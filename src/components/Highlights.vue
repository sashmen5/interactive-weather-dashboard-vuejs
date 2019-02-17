<template>
  <div class="custom-content-card content-card card">
    <div class="card-body pb-0">
      <div class="content-header h4 text-center pt-2 pb-3">Highlights</div>
      <div class="row">
        <uv-index :highlights="highlights"></uv-index>
        <visibility :highlights="highlights"></visibility>
        <wind-status :highlights="highlights"></wind-status>
      </div>
    </div>
  </div>
</template>

<script>
  import UVIndex from "./UVIndex.vue";
  import Visibility from "./Visibility.vue";
  import WindStatus from "./WindStatus.vue";

  export default {
    props: ["highlights"],
    components: {
      "uv-index": UVIndex,
      "visibility": Visibility,
      "wind-status": WindStatus,
    },
  };
</script>
