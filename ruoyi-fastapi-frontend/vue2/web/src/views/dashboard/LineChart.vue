<style lang="scss" scoped></style>

<template>
  <div
    :class="className"
    :style="{ height: height, width: width }"/>
</template>

<script>
/**
 * 组件名称：LineChart
 * 功能说明：首页折线图组件，基于 ECharts 展示期望与实际数据走势
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String}  className  - 图表容器类名
 * @prop {String}  width      - 图表宽度
 * @prop {String}  height     - 图表高度
 * @prop {Boolean} autoResize - 是否自适应尺寸
 * @prop {Object}  chartData  - 图表数据
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
    height: { type: String, default: '350px' },
    autoResize: { type: Boolean, default: true },
    chartData: { type: Object, required: true }
  },

  data()
  {
    return {
      chart: null,
    }
  },

  watch:
  {
    chartData:
    {
      deep: true,
      /**
       * 监听图表数据变化并刷新配置
       * @param {Object} val - 新的图表数据
       */
      handler(val)
      {
        this.setOptions(val)
      },
    },
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
     * 初始化图表并设置折线图配置项
     */
    initChart()
    {
      this.chart = echarts.init(this.$el, 'macarons')
      this.setOptions(this.chartData)
    },

    /**
     * 设置折线图配置项
     * @param {Object} options              - 图表数据
     * @param {Array}  options.expectedData - 期望数据
     * @param {Array}  options.actualData   - 实际数据
     */
    setOptions({ expectedData, actualData } = {})
    {
      this.chart.setOption({
        xAxis: {
          data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
          boundaryGap: false,
          axisTick: {
            show: false,
          },
        },
        grid: {
          left: 10,
          right: 10,
          bottom: 20,
          top: 30,
          containLabel: true,
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
          },
          padding: [5, 10],
        },
        yAxis: {
          axisTick: {
            show: false,
          },
        },
        legend: {
          data: ['expected', 'actual'],
        },
        series: [
          {
            name: 'expected',
            itemStyle: {
              normal: {
                color: '#FF005A',
                lineStyle: {
                  color: '#FF005A',
                  width: 2,
                },
              },
            },
            smooth: true,
            type: 'line',
            data: expectedData,
            animationDuration: 2800,
            animationEasing: 'cubicInOut',
          },
          {
            name: 'actual',
            smooth: true,
            type: 'line',
            itemStyle: {
              normal: {
                color: '#3888fa',
                lineStyle: {
                  color: '#3888fa',
                  width: 2,
                },
                areaStyle: {
                  color: '#f3f8ff',
                },
              },
            },
            data: actualData,
            animationDuration: 2800,
            animationEasing: 'quadraticOut',
          },
        ],
      })
    },
  },
}
</script>