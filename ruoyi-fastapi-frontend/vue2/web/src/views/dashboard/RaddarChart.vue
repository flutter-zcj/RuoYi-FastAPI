<style lang="scss" scoped></style>

<template>
  <div
    :class="className"
    :style="{ height: height, width: width }"/>
</template>

<script>
/**
 * 组件名称：RaddarChart
 * 功能说明：首页雷达图组件，基于 ECharts 展示预算与支出对比
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

const animationDuration = 3000

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
     * 初始化图表并设置雷达图配置项
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
        radar: {
          radius: '66%',
          center: ['50%', '42%'],
          splitNumber: 8,
          splitArea: {
            areaStyle: {
              color: 'rgba(127,95,132,.3)',
              opacity: 1,
              shadowBlur: 45,
              shadowColor: 'rgba(0,0,0,.5)',
              shadowOffsetX: 0,
              shadowOffsetY: 15,
            },
          },
          indicator: [
            { name: 'Sales', max: 10000 },
            { name: 'Administration', max: 20000 },
            { name: 'Information Techology', max: 20000 },
            { name: 'Customer Support', max: 20000 },
            { name: 'Development', max: 20000 },
            { name: 'Marketing', max: 20000 },
          ],
        },
        legend: {
          left: 'center',
          bottom: '10',
          data: ['Allocated Budget', 'Expected Spending', 'Actual Spending'],
        },
        series: [
          {
            type: 'radar',
            symbolSize: 0,
            areaStyle: {
              normal: {
                shadowBlur: 13,
                shadowColor: 'rgba(0,0,0,.2)',
                shadowOffsetX: 0,
                shadowOffsetY: 10,
                opacity: 1,
              },
            },
            data: [
              {
                value: [5000, 7000, 12000, 11000, 15000, 14000],
                name: 'Allocated Budget',
              },
              {
                value: [4000, 9000, 15000, 15000, 13000, 11000],
                name: 'Expected Spending',
              },
              {
                value: [5500, 11000, 12000, 15000, 12000, 12000],
                name: 'Actual Spending',
              },
            ],
            animationDuration: animationDuration,
          },
        ],
      })
    },
  },
}
</script>