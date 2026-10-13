<style lang="scss" scoped></style>

<template>
  <div
    :class="className"
    :style="{ height: height, width: width }"/>
</template>

<script>
/**
 * 组件名称：PieChart
 * 功能说明：首页饼图组件，基于 ECharts 展示各行业周文章占比
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} className - 图表容器类名
 * @prop {String} width     - 图表宽度
 * @prop {String} height    - 图表高度
 *
 * 变量说明：
 * @data {Object} chart - ECharts 实例
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'

export default
{
  mixins: [resize],

  props:
  {
    className: { type: String, default: 'chart' },
    width: { type: String, default: '100%' },
    height: { type: String, default: '300px' }
  },

  data()
  {
    return {
      chart: null,
    }
  },

  mounted()
  {
    this.$nextTick(() =>
    {
      this.initChart()
    })
  },

  beforeDestroy()
  {
    if (!this.chart)
    {
      return
    }
    this.chart.dispose()
    this.chart = null
  },

  methods:
  {
    /**
     * 初始化图表并设置饼图配置项
     */
    initChart()
    {
      this.chart = echarts.init(this.$el, 'macarons')

      this.chart.setOption({
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)',
        },
        legend: {
          left: 'center',
          bottom: '10',
          data: ['Industries', 'Technology', 'Forex', 'Gold', 'Forecasts'],
        },
        series: [
          {
            name: 'WEEKLY WRITE ARTICLES',
            type: 'pie',
            roseType: 'radius',
            radius: [15, 95],
            center: ['50%', '38%'],
            data: [
              { value: 320, name: 'Industries' },
              { value: 240, name: 'Technology' },
              { value: 149, name: 'Forex' },
              { value: 100, name: 'Gold' },
              { value: 59, name: 'Forecasts' },
            ],
            animationEasing: 'cubicInOut',
            animationDuration: 2600,
          },
        ],
      })
    },
  },
}
</script>