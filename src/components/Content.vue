<template>
  <div id="pagecontent">
    <p>This child components of Content.vue are:</p>
    <ul>
      <li v-for="child in childComponents">{{ child }}</li>
    </ul>
    {{ weather_data }}
    <temp-var-chart :tempVar="tempVar"></temp-var-chart>
    <today-highlights :highlights="highlights"></today-highlights>
  </div>
</template>
<script>
  import TempVarChart from './TempVarChart.vue'
  import Highlights from './Highlights.vue'

	export default {
		name: "Content",
    components: {
      'temp-var-chart': TempVarChart,
      'today-highlights': Highlights
    },
    props: ["weather_data"],
    data() {
		  return {
		    childComponents: ['TempVarChart.vue', 'Highlights.vue'],
        tempVar: this.weather_data.temperature,
        highlights: this.weather_data.highlights,
      }
    }
	}
</script>

<style scoped>


</style>
