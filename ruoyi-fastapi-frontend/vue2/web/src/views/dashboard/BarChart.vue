<style lang="scss" scoped></style>

<template>
  <div
    :class="className"
    :style="{ height: height, width: width }"/>
</template>

<script>
/**
 * 组件名称：BarChart
 * 功能说明：首页柱状图组件，基于 ECharts 展示不同页面访问量
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

const animationDuration = 6000

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
     * 初始化图表并设置柱状图配置项
     */
    initChart()
    {
      this.chart = echarts.init(this.$el, 'macarons')

      this.chart.setOption({
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            // 坐标轴指示器，坐标轴触发有效
            type: 'shadow', // 默认为直线，可选为：'line' | 'shadow'
          },
        },
        grid: {
          top: 10,
          left: '2%',
          right: '2%',
          bottom: '3%',
          containLabel: true,
        },
        xAxis: [
          {
            type: 'category',
            data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
            axisTick: {
              alignWithLabel: true,
            },
          },
        ],
        yAxis: [
          {
            type: 'value',
            axisTick: {
              show: false,
            },
          },
        ],
        series: [
          {
            name: 'pageA',
            type: 'bar',
            stack: 'vistors',
            barWidth: '60%',
            data: [79, 52, 200, 334, 390, 330, 220],
            animationDuration,
          },
          {
            name: 'pageB',
            type: 'bar',
            stack: 'vistors',
            barWidth: '60%',
            data: [80, 52, 200, 334, 390, 330, 220],
            animationDuration,
          },
          {
            name: 'pageC',
            type: 'bar',
            stack: 'vistors',
            barWidth: '60%',
            data: [30, 52, 200, 334, 390, 330, 220],
            animationDuration,
          },
        ],
      })
    },
  },
}
</script>