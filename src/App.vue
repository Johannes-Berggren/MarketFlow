<template>
  <div class="wrapper">
    <chart class="stock" :constructor-type="'stockChart'" :options="chartData" />
    <a href="https://johannesberggren.com">JohannesBerggren.com</a>
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
              name: 'VIX High',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'VIX Low',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'ISM',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'AAII Bulls minus Bears',
              data: [],
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

      // console.log(Date.UTC(2005, 1, 3))

      fetch('./vix-daily_csv.csv')
        .then(response => response.text())
        .then(VIXRawData => {
          Papa.parse(VIXRawData, {
            complete: function (results) {
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[0].data.push([
                  new Date(results.data[i][0]).getTime(),
                  parseFloat(results.data[i][2])
                ])
                self.chartData.series[1].data.push([
                  new Date(results.data[i][0]).getTime(),
                  parseFloat(results.data[i][3])
                ])
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
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[2].data.push([
                  new Date(results.data[i][0]).getTime(),
                  parseFloat(results.data[i][1])
                ])
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
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[3].data.push([
                  new Date(results.data[i][0]).getTime(),
                  parseFloat(results.data[i][1] - results.data[i][3]) * 100 + 50
                ])
              }
            }
          })
        })
    }
  }
</script>

<style scoped>
  a {
    float: right;
    font-size: 12px;
  }

  .stock {
    width: 100%;
    margin: 0 auto
  }
</style>
