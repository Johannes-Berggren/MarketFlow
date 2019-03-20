<template>
  <el-container>
    <el-header>
      <el-row type="flex" justify="center">
        <h1>MarketFlow</h1>
      </el-row>
    </el-header>
    <el-main>
      <chart class="stock" v-loading="!(l1 && l2 && l3 && l4)" :constructor-type="'stockChart'" :options="chartData" />
      <el-row style="text-align: center">
        <h2 style="color: grey"><i>Is now the time to go all stocks, or all cash?</i></h2>
      </el-row>
      <el-row :gutter="40">
        <el-col :xs="24" :sm="12">
          <h2><span style="color: #6df06e; font-size: 70px">•</span> ISM - PMI Composite Index</h2>
          <p>The PMI (R) is a composite index based on the diffusion indexes of five of the indexes with equal weights:
             New
             Orders (seasonally adjusted), Production (seasonally adjusted), Employment (seasonally adjusted), Supplier
             Deliveries (seasonally adjusted), and Inventories.</p>
          <p class="data-source"><i>Data from Quandl.com</i></p>
        </el-col>
        <el-col :xs="24" :sm="12">
          <h2><span style="color: #3a8ee6; font-size: 70px">•</span> VIX - CBOE Volatility Index</h2>
          <p>VIX measures market expectation of near term volatility conveyed by stock index option prices. The original
             VIX
             was constructed using the implied volatilities of eight different OEX option series so that, at any given
             time,
             it represented the implied volatility of a hypothetical at-the-money OEX option with exactly 30 days to
             expiration.</p>
          <p class="data-source"><i>Data from Datahub.io</i></p>
        </el-col>
        <el-col :xs="24" :sm="12">
          <h2><span style="color: #ff9f4c; font-size: 70px">•</span> AAII - Investor Sentiment Data</h2>
          <p>The AAII Investor Sentiment Survey measures the percentage of individual investors who are bullish,
             bearish,
             and neutral on the stock market for the next six months; individuals are polled from the ranks of the AAII
             membership on a weekly basis. Only one vote per member is accepted in each weekly voting period.</p>
          <p class="data-source"><i>Data from Quandl.com</i></p>
        </el-col>
        <el-col :xs="24" :sm="12">
          <h2><span style="color: #8082f0; font-size: 70px">•</span> Eurodollar Futures</h2>
          <p>Historical Futures Prices: Eurodollar Futures, Continuous Contract #1. Non-adjusted price based on
             spot-month
             continuous contract calculations. Raw data from CME.</p>
          <p class="data-source"><i>Data from Quandl.com</i></p>
        </el-col>
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
          chart: {
            height: '50%'
          },
          xAxis: {
            min: new Date().getTime() - (1000 * 3600 * 24 * 365),
            max: new Date().getTime()
          },
          series: [
            {
              name: 'VIX Low',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'VIX High',
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
            },
            {
              name: 'Eurodollar Futures',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            },
            {
              name: 'S&P 500 Weekly High',
              data: [],
              tooltip: {
                valueDecimals: 2
              }
            }
          ]
        },
        l1: false,
        l2: false,
        l3: false,
        l4: false
      }
    },
    mounted () {
      const self = this

      fetch('https://datahub.io/core/finance-vix/r/vix-daily.csv')
        .then(response => response.text())
        .then(VIXRawData => {
          Papa.parse(VIXRawData, {
            complete: function (results) {
              let lowMax  = parseFloat(results.data[1][3]),
                  lowMin  = parseFloat(results.data[1][3]),
                  highMax = parseFloat(results.data[1][2]),
                  highMin = parseFloat(results.data[1][2])

              for (let i = 1; i < results.data.length - 1; i++) {
                let lowVal  = parseFloat(results.data[i][3]),
                    highVal = parseFloat(results.data[i][2])

                if (lowVal > lowMax) lowMax = lowVal
                if (lowVal < lowMin) lowMin = lowVal
                if (highVal > highMax) highMax = highVal
                if (highVal < highMin) highMin = highVal
              }

              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[1].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat(results.data[i][3]), lowMax, lowMin)
                ])
                self.chartData.series[0].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat(results.data[i][2]), highMax, highMin)
                ])
              }
              self.l1 = true
            }
          })
        })

      fetch('https://www.quandl.com/api/v3/datasets/ISM/MAN_PMI.csv?api_key=-mk6d2xWHxz3vdHg6fQj')
        .then(response => response.text())
        .then(ISMRawData => {
          Papa.parse(ISMRawData, {
            complete: function (results) {
              let max = parseFloat(results.data[1][1]),
                  min = parseFloat(results.data[1][1])

              for (let i = 1; i < results.data.length - 1; i++) {
                let val = parseFloat(results.data[i][1])

                if (val > max) max = val
                if (val < min) min = val
              }

              results.data = results.data.reverse()
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[2].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat(results.data[i][1]), max, min)
                ])
              }
              self.l2 = true
            }
          })
        })

      fetch('https://www.quandl.com/api/v3/datasets/AAII/AAII_SENTIMENT.csv?api_key=-mk6d2xWHxz3vdHg6fQj')
        .then(response => response.text())
        .then(AAIIRawData => {
          Papa.parse(AAIIRawData, {
            complete: function (results) {
              let aaiiMax = parseFloat((results.data[1][5])),
                  aaiiMin = parseFloat((results.data[1][5])),
                  snpMax = parseFloat((results.data[1][10])),
                  snpMin = parseFloat((results.data[1][10]))

              for (let i = 1; i < results.data.length - 1; i++) {
                let aaiiVal = parseFloat((results.data[i][5])),
                    snpVal = parseFloat(results.data[i][10])

                if (aaiiVal > aaiiMax) aaiiMax = aaiiVal
                if (aaiiVal < aaiiMin) aaiiMin = aaiiVal
                if (snpVal > snpMax) snpMax = snpVal
                if (snpVal < snpMin) snpMin = snpVal
              }

              results.data = results.data.reverse()
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[3].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat((results.data[i][5])), aaiiMax, aaiiMin)
                ])
                self.chartData.series[5].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat(results.data[i][10]), snpMax, snpMin)
                ])
              }
              self.l3 = true
            }
          })
        })

      fetch('https://www.quandl.com/api/v3/datasets/CHRIS/CME_ED1.csv?api_key=-mk6d2xWHxz3vdHg6fQj')
        .then(response => response.text())
        .then(EuroDollar => {
          Papa.parse(EuroDollar, {
            complete: function (results) {
              let max = parseFloat(results.data[1][1]),
                  min = parseFloat(results.data[1][1])

              for (let i = 1; i < results.data.length - 1; i++) {
                let val = parseFloat(results.data[i][1])

                if (val > max) max = val
                if (val < min) min = val
              }

              results.data = results.data.reverse()
              for (let i = 1; i < results.data.length - 1; i++) {
                self.chartData.series[4].data.push([
                  new Date(results.data[i][0]).getTime(),
                  self.normalize(parseFloat(results.data[i][1]), max, min)
                ])
              }
              self.l4 = true
            }
          })
        })
    },
    methods: {
      normalize (val, max, min) {
        return (val - min) / (max - min)
      }
    }
  }
</script>

<style scoped>
  h1 {
    font-size: 60px;
    margin: 0 40px;
    color: grey;
  }

  a {
    float: right;
    font-size: 12px;
  }

  .stock {
    width: 100%;
    margin: 0 auto 50px;
  }

  .data-source {
    color: grey;
    font-size: 10px;
  }
</style>
