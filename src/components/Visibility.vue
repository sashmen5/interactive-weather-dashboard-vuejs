<template>
  <div class="highlights-item col-md-4 col-sm-6 col-xs-12 border-left border-right border-top">
    <div>
      <fusioncharts
        :type="type"
        :width="width"
        :height="height"
        :containerbackgroundopacity="containerbackgroundopacity"
        :dataformat="dataformat"
        :datasource="datasource"
      >
      </fusioncharts>
    </div>
  </div>
</template>

<script>
  export default {
    props: ["highlights"],
    components: {},
    methods: {},
    computed: {},
    data() {
      return {
        type: "hlineargauge",
        width: "100%",
        height: "100%",
        containerbackgroundopacity: 0,
        dataformat: "json",
        creditLabel: false,
        datasource: {
          chart: {
            caption: "Air Visibility",
            captionFontBold: "0",
            captionFontColor: "#000000",
            baseFont: "Roboto",
            numberSuffix: " km",
            lowerLimit: "0",
            upperLimit: "40",
            showPointerShadow: "1",
            animation: "1",
            transposeAnimation: "1",
            theme: "fusion",
            bgAlpha: "0",
            canvasBgAlpha: "0",
            valueFontSize: "20",
            valueFontColor: "#000000",
            valueFontBold: "1",
            pointerBorderAlpha: "0",
            chartBottomMargin: "40",
            captionPadding: "30",
            chartTopMargin: "30"
          },
          colorRange: {
            color: [
              {
                minValue: "0",
                maxValue: "4",
                label: "Fog",
                code: "#6297d9"
              },
              {
                minValue: "4",
                maxValue: "10",
                label: "Haze",
                code: "#7DA9E0"
              },
              {
                minValue: "10",
                maxValue: "40",
                label: "Clear",
                code: "#D8EDFF"
              }
            ]
          },
          pointers: {
            pointer: [
              {
                value: this.highlights.visibility.toString()
              }
            ]
          }
        }
      };
    },
    watch: {
      highlights: {
        handler: function() {
          this.datasource.pointers.pointer[0].value = this.highlights.visibility.toString();
        },
        deep: true
      }
    }
  };
</script>
