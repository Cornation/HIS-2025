<template>
  <div :class="className" :style="{height: height, width: width, border: '1px solid #eee'}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'
import { deptClassifyStatistics } from '@/api/charts'

const animationDuration = 3000

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      data: {
        dateOfSevenDays: [],
        dispositionAmount: [],
        checkAmount: [],
        testAmount: [],
        medicineAmount: [],
        herbalAmount: []
      },
      chart: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.fetchChartData()
    })
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose()
      this.chart = null
    }
  },
  methods: {
    fetchChartData() {
      deptClassifyStatistics(this.$store.getters.deptId).then(res => {
        if (res && res.data) {
          this.data = {
            dateOfSevenDays: res.data.dateOfSevenDays || [],
            dispositionAmount: res.data.dispositionAmount || [],
            checkAmount: res.data.checkAmount || [],
            testAmount: res.data.testAmount || [],
            medicineAmount: res.data.medicineAmount || [],
            herbalAmount: res.data.herbalAmount || []
          }
          this.initChart()
        }
      })
      // this.initChart()
      .catch(err => {
        // console.log(url)
        console.error('获取科室分类费用数据失败:', err)
      })
    },
    initChart() {
      if (!this.chart) {
        this.chart = echarts.init(this.$el, 'macarons')
      }

      this.chart.setOption({
        title: {
          left: 'center',
          text: '科室分类费用统计',
          textStyle: {
            fontSize: 16
          }
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        grid: {
          top: 30,
          left: '2%',
          right: '2%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [{
          type: 'category',
          data: this.data.dateOfSevenDays,
          axisTick: {
            alignWithLabel: true
          }
        }],
        yAxis: [{
          type: 'value',
          axisTick: {
            show: false
          }
        }],
        series: [
          {
            name: '处置费',
            type: 'bar',
            stack: 'vistors',
            barWidth: '30%',
            // data: [100, 120, 130, 90, 80, 70, 60],
            data: this.data.dispositionAmount,
            animationDuration
          },
          {
            name: '检查费',
            type: 'bar',
            stack: 'vistors',
            barWidth: '30%',
            // data: [100, 120, 130, 90, 80, 70, 60],
            data: this.data.checkAmount,
            animationDuration
          },
          {
            name: '检验费',
            type: 'bar',
            stack: 'vistors',
            barWidth: '30%',
            // data: [100, 120, 130, 90, 80, 70, 60],
            data: this.data.testAmount,
            animationDuration
          },
          {
            name: '西药费',
            type: 'bar',
            stack: 'vistors',
            barWidth: '30%',
            // data: [100, 120, 130, 90, 80, 70, 60],
            data: this.data.medicineAmount,
            animationDuration
          },
          {
            name: '中药费',
            type: 'bar',
            stack: 'vistors',
            barWidth: '30%',
            // data: [100, 120, 130, 90, 80, 70, 60],
            data: this.data.herbalAmount,
            animationDuration
          }
        ]
      })
    }
  }
}
</script>
