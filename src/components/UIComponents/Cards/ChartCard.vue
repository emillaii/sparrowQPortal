<template>
  <div class="card">
    <div class="card-header" v-if="$slots.header">
      <slot name="header"></slot>
    </div>
    <div class="card-body">
      <div :id="chartId" class="ct-chart"></div>
    </div>
    <div class="card-footer" v-if="$slots.footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>
<script>
  import Card from './Card.vue'
  import EventBus from '../../../event-bus.js'
  import {elasticSearchBaseUrl} from '../../../globalConstants.js'

  export default {
    name: 'chart-card',
    components: {
      Card
    },
    props: {
      chartType: {
        type: String,
        default: 'Line' // Line | Pie | Bar
      },
      chartOptions: {
        type: Object,
        default: () => {
          return {}
        }
      },
      chartData: {
        type: Object,
        default: () => {
          return {
            labels: [],
            series: []
          }
        }
      },
      sfrUrl: '',
      responsiveOptions: [Object, Array]
    },
    data () {
      return {
        timer: '',
        chartId: 'no-id',
        $Chartist: null,
        chart: null
      }
    },
    created() {
        //this.fetchData()
        EventBus.$on('selected-unit-id', function(unitId) {
          var url = 'http://104.199.174.240:8000/sfrlog/_doc/' + unitId
          this.fetchData(url)
        }.bind(this));
    },
    methods: {
      /***
       * Initializes the chart by merging the chart options sent via props and the default chart options
       http://104.199.174.240:8000/sfrlog/_doc/rlzm-mUBCE2ArQqZuCpM
       */
      fetchData (url) {
        if (this.chartType == "Line") {
          this.$http.get(url).then(response => {
            this.someData = response.body
            var ccData_x = [], ccData_y = []
            var ulData_x = [], ulData_y = []
            var urData_x = [], urData_y = []
            var llData_x = [], llData_y = []
            var lrData_x = [], lrData_y = []
            for (var i in response.body._source.CC) {
              ccData_x.push(Math.round(response.body._source.CC[i].pz*10000)/10000)
              ccData_y.push(response.body._source.CC[i].sfr)
              if (response.body._source.UL.length >= i) ulData_y.push(response.body._source.UL[i].sfr)
              urData_y.push(response.body._source.UR[i].sfr)
              llData_y.push(response.body._source.LL[i].sfr)
              lrData_y.push(response.body._source.LR[i].sfr)
            }
            this.chartData.labels = ccData_x
            this.chartData.series = []
            this.chartData.series.push(ccData_y)
            this.chartData.series.push(ulData_y)
            this.chartData.series.push(urData_y)
            this.chartData.series.push(llData_y)
            this.chartData.series.push(lrData_y)
            if (this.chart != undefined) this.chart.update()
          }, response => {
            //error callback
          });
        }
      },
      initChart () {
        var chartIdQuery = `#${this.chartId}`
        this.chart = this.$Chartist[this.chartType](chartIdQuery, this.chartData, this.chartOptions, this.responsiveOptions)
        this.$emit('initialized', this.chart)
        if (this.chartType === 'Line') {
          this.animateLineChart()
        }
        if (this.chartType === 'Bar') {
          this.animateBarChart()
        }
      },
      /***
       * Assigns a random id to the chart
       */
      updateChartId () {
        const currentTime = new Date().getTime().toString()
        const randomInt = this.getRandomInt(0, currentTime)
        this.chartId = `div_${randomInt}`
      },
      getRandomInt (min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min
      },
      animateLineChart () {
        let seq = 0
        let durations = 500
        let delays = 80
        this.chart.on('draw', (data) => {
          if (data.type === 'line' || data.type === 'area') {
            data.element.animate({
              d: {
                begin: 600,
                dur: 700,
                from: data.path.clone().scale(1, 0).translate(0, data.chartRect.height()).stringify(),
                to: data.path.clone().stringify(),
                easing: this.$Chartist.Svg.Easing.easeOutQuint
              }
            })
          } else if (data.type === 'point') {
            seq++
            data.element.animate({
              opacity: {
                begin: seq * delays,
                dur: durations,
                from: 0,
                to: 1,
                easing: 'ease'
              }
            })
          }
        })
        seq = 0
      },
      animateBarChart () {
        let seq = 0
        let durations = 500
        let delays = 80
        this.chart.on('draw', (data) => {
          if (data.type === 'bar') {
            seq++
            data.element.animate({
              opacity: {
                begin: seq * delays,
                dur: durations,
                from: 0,
                to: 1,
                easing: 'ease'
              }
            })
          }
        })
      }
    },
    async mounted () {
      this.updateChartId()
      const Chartist = await import('chartist')
      this.$Chartist = Chartist
      this.initChart()
    }
  }
</script>
<style>

</style>
