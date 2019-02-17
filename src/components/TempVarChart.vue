<template>
  <div class="custom-card header-card card">
    <div class="card-body pt-0">
      <fusioncharts
        type="spline"
        width="100%"
        height="100%"
        dataFormat="json"
        dataEmptyMessage="i-https://i.postimg.cc/R0QCk9vV/Rolling-0-9s-99px.gif"
        :dataEmptyMessageImageScale="dataEmptyMessageImageScale"
        :dataSource="tempChartData"
      >
      </fusioncharts>
    </div>

  </div>
</template>

<script>
  export default {
    props: ["tempVar"],
    components: {},
    data() {
      return {
        dataEmptyMessageImageScale: 39,
        tempChartData: {
          chart: {
            caption: "Hourly Temperature",
            captionFontBold: "0",
            captionFontColor: "#000000",
            captionPadding: "30",
            baseFont: "Roboto",
            chartTopMargin: "30",
            showHoverEffect: "1",
            theme: "fusion",
            showaxislines: "1",
            numberSuffix: "°C",
            anchorBgColor: "#6297d9",
            paletteColors: "#6297d9",
            drawCrossLine: "1",
            plotToolText: "$label<br><hr><b>$dataValue</b>",
            showAxisLines: "0",
            showYAxisValues: "0",
            anchorRadius: "4",
            divLineAlpha: "0",
            labelFontSize: "13",
            labelAlpha: "65",
            labelFontBold: "0",
            rotateLabels: "1",
            slantLabels: "1",
            canvasPadding: "20"
          },
          data: [],
        },
      };
    },
    methods: {
      setChartData: function() {
        var data = [];
        for (var i = 0; i < this.tempVar.tempToday.length; i++) {
          var dataObject = {
            label: this.tempVar.tempToday[i].hour,
            value: this.tempVar.tempToday[i].temp
          };
          data.push(dataObject);
        }
        this.tempChartData.data = data;
      },
    },
    mounted: function() {
      this.setChartData();
    },
    watch: {
      tempVar: {
        handler: function() {
          this.setChartData();
        },
        deep: true
      },
    },
  };
</script>

<style>
</style>
