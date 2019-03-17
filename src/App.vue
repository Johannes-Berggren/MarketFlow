<template>
  <el-container>
    <el-header>
      <el-row type="flex" justify="center">
        <h1>MarketFlow</h1>
      </el-row>
    </el-header>
    <el-main>
      <chart class="stock" :constructor-type="'stockChart'" :options="chartData" />
      <el-row>
        <h2>ISM - PMI Composite Index</h2>
        <p>The PMI (R) is a composite index based on the diffusion indexes of five of the indexes with equal weights:
           New
           Orders (seasonally adjusted), Production (seasonally adjusted), Employment (seasonally adjusted), Supplier
           Deliveries (seasonally adjusted), and Inventories.</p>
        <h2>VIX - CBOE Volatility Index</h2>
        <p>VIX measures market expectation of near term volatility conveyed by stock index option prices. The original
           VIX
           was constructed using the implied volatilities of eight different OEX option series so that, at any given
           time,
           it represented the implied volatility of a hypothetical at-the-money OEX option with exactly 30 days to
           expiration.</p>
        <h2>AAII - Investor Sentiment Data (Bullish - Bearish + 50)</h2>
        <p>The AAII Investor Sentiment Survey measures the percentage of individual investors who are bullish, bearish,
           and neutral on the stock market for the next six months; individuals are polled from the ranks of the AAII
           membership on a weekly basis. Only one vote per member is accepted in each weekly voting period.</p>
      </el-row>
    </el-main>
    <el-footer>
      <a href="https://johannesberggren.com">JohannesBerggren.com</a>
    </el-footer>
  </el-container>
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
          xAxis: {
            min: new Date().getTime() - (1000 * 3600 * 24 * 365),
            max: new Date().getTime()
          },
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

      fetch('https://datahub.io/core/finance-vix/r/vix-daily.csv')
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

      fetch('https://www.quandl.com/api/v3/datasets/ISM/MAN_PMI.csv?api_key=-mk6d2xWHxz3vdHg6fQj')
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

      fetch('https://www.quandl.com/api/v3/datasets/AAII/AAII_SENTIMENT.csv?api_key=-mk6d2xWHxz3vdHg6fQj')
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
