<template>
  <div class="wrapper">
    <!--<chartist-->
    <!--:data="chartData"-->
    <!--ratio="ct-major-second"-->
    <!--type="Line"-->
    <!--:options="{-->
    <!--showArea: true,-->
    <!--showLine: false,-->
    <!--showPoint: false,-->
    <!--fullWidth: true,-->
    <!--axisX: {-->
    <!--showLabel: true,-->
    <!--showGrid: false-->
    <!--}-->
    <!--}">-->
    <!--</chartist>-->
    <chart class="stock" :constructor-type="'stockChart'" :options="chartData" />
    <!--<pre>{{ chartData.series }}</pre>-->
  </div>
</template>

<script>
  import { Chart }  from 'highcharts-vue'
  import Highcharts from 'highcharts'
  import stockInit  from 'highcharts/modules/stock'
  import Papa       from 'papaparse'

  stockInit(Highcharts)


  export default {
    components: {
      Chart
    },
    data () {
      return {
        chartData: {
          labels: [],
          series: [
            {
              name: 'VIX Open',
              data: [],
              pointStart: Date.UTC(2005, 1, 3), // FIXME
              pointInterval: 1000 * 3600 * 33, // FIXME: Doesn't account for weekends
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'VIX High',
              data: [],
              pointStart: Date.UTC(2005, 1, 3), // FIXME
              pointInterval: 1000 * 3600 * 33, // FIXME: Doesn't account for weekends
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'VIX Low',
              data: [],
              pointStart: Date.UTC(2005, 1, 3), // FIXME
              pointInterval: 1000 * 3600 * 33, // FIXME: Doesn't account for weekends
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'VIX Close',
              data: [],
              pointStart: Date.UTC(2005, 1, 3), // FIXME
              pointInterval: 1000 * 3600 * 33, // FIXME: Doesn't account for weekends
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'ISM',
              data: [],
              pointStart: Date.UTC(2005, 1, 1), // FIXME
              pointInterval: 1000 * 3600 * 24 * 30, // FIXME: Doesn't account for weekends
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'AAII Bullish',
              data: [],
              pointStart: Date.UTC(2005, 1, 6), // FIXME
              pointInterval: 1000 * 3600 * 24 * 7,
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'AAII Neutral',
              data: [],
              pointStart: Date.UTC(2005, 1, 6), // FIXME
              pointInterval: 1000 * 3600 * 24 * 7,
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'AAII Bearish',
              data: [],
              pointStart: Date.UTC(2005, 1, 6), // FIXME
              pointInterval: 1000 * 3600 * 24 * 7,
              tooltip: {
                valueDecimals: 2
              }
            }
          ]
        },
        VIXHigh: [],
        VIXLow: []
      }
    },
    mounted () {
      const self = this

      fetch('./vix-daily_csv.csv')
        .then(response => response.text())
        .then(VIXRawData => {
          Papa.parse(VIXRawData, {
            complete: function (results) {
              for (let i = 1; i < results.data.length; i++) {
                // self.chartData.series[0].data.push(parseFloat(results.data[i][1]))
                self.chartData.series[1].data.push(parseFloat(results.data[i][2]))
                self.chartData.series[2].data.push(parseFloat(results.data[i][3]))
                // self.chartData.series[3].data.push(parseFloat(results.data[i][4]))
              }
            }
          })
        })

      fetch('./ISM-MAN_PMI.csv')
        .then(response => response.text())
        .then(ISMRawData => {
          Papa.parse(ISMRawData, {
            complete: function (results) {
              results.data = results.data.reverse()
              for (let i = 1; i < results.data.length; i++) {
                self.chartData.series[4].data.push(parseFloat(results.data[i][1]))
              }
            }
          })
        })

      fetch('./AAII-AAII_SENTIMENT.csv')
        .then(response => response.text())
        .then(AAIIRawData => {
          Papa.parse(AAIIRawData, {
            complete: function (results) {
              results.data = results.data.reverse()
              for (let i = 1; i < results.data.length; i++) {
                self.chartData.series[5].data.push(parseFloat(results.data[i][1]) * 100)
                // self.chartData.series[6].data.push(parseFloat(results.data[i][2]) * 100)
                self.chartData.series[7].data.push(parseFloat(results.data[i][3]) * 100)
              }
            }
          })
        })
    }
  }
</script>

<style scoped>
  .stock {
    width: 100%;
    margin: 0 auto
  }
</style>
