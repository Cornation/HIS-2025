<template>
  <div :class="className" :style="{ height: height, width: width }" />
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // 需要确保 macarons 存在
import resize from './mixins/resize'
import { staffClassifyStatistics } from '@/api/charts'

export default {
  mixins: [resize],
  props: {
    className: { type: String, default: 'chart' },
    width: { type: String, default: '100%' },
    height: { type: String, default: '300px' }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.loadData()
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose()
    }
    this.chart = null
  },
  methods: {
    async loadData() {
      try {
        const res = await staffClassifyStatistics(this.$store.getters.id)
        const { amountCat = [], amount = [] } = res.data || {}

        if (!amountCat.length || !amount.length) {
          console.warn("📭 没有图表数据")
          return
        }

        const chartData = amountCat.map((name, i) => ({
          name,
          value: amount[i] || 0
        }))

        this.chart = echarts.init(this.$el, 'macarons')
        this.chart.setOption({
          tooltip: {
            trigger: 'item',
            formatter: '{a} <br/>{b} : {c} ({d}%)'
          },
          legend: {
            left: 'center',
            bottom: '10',
            data: amountCat
          },
          title: {
            left: 'left',
            text: '七日分类费用统计',
            fontSize: 1
          },
          series: [
            {
              name: '分类费用统计',
              type: 'pie',
              roseType: 'radius',
              radius: [15, 95],
              center: ['50%', '38%'],
              data: chartData,
              animationEasing: 'cubicInOut',
              animationDuration: 2600
            }
          ]
        })
      } catch (e) {
        console.error("❌ 图表数据加载失败", e)
      }
    }
  }
}
</script>
