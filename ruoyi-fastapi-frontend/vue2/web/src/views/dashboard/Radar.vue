<style lang="scss" scoped></style>

<template>
  <v-chart
    :forceFit="true"
    height="400"
    :data="data"
    :padding="[20, 20, 95, 20]"
    :scale="scale">
    <v-tooltip></v-tooltip>
    <v-axis
      :dataKey="axis1Opts.dataKey"
      :line="axis1Opts.line"
      :tickLine="axis1Opts.tickLine"
      :grid="axis1Opts.grid"/>
    <v-axis
      :dataKey="axis2Opts.dataKey"
      :line="axis2Opts.line"
      :tickLine="axis2Opts.tickLine"
      :grid="axis2Opts.grid"/>
    <v-legend
      dataKey="user"
      marker="circle"
      :offset="30"/>
    <v-coord
      type="polar"
      radius="0.8"/>
    <v-line
      position="item*score"
      color="user"
      :size="2"/>
    <v-point
      position="item*score"
      color="user"
      :size="4"
      shape="circle"/>
  </v-chart>
</template>

<script>
/**
 * 组件名称：Radar
 * 功能说明：团队指数雷达图组件，基于 viser-vue 绘制
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Array} data - 雷达图数据
 *
 * 变量说明：
 * @data {Object} axis1Opts - 第一组坐标轴配置
 * @data {Object} axis2Opts - 第二组坐标轴配置
 * @data {Array}  scale     - 数据度量配置
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import Viser from 'viser-vue'
import Vue from 'vue'

Vue.use(Viser)

const axis1Opts = {
  dataKey: 'item',
  line: null,
  tickLine: null,
  grid: {
    lineStyle: {
      lineDash: null,
    },
    hideFirstLine: false,
  },
}
const axis2Opts = {
  dataKey: 'score',
  line: null,
  tickLine: null,
  grid: {
    type: 'polygon',
    lineStyle: {
      lineDash: null,
    },
  },
}

const scale = [
  {
    dataKey: 'score',
    min: 0,
    max: 80,
  },
  {
    dataKey: 'user',
    alias: '类型',
  },
]

export default
{
  name: 'Radar',

  props:
  {
    data: { type: Array, default: null }
  },

  data()
  {
    return {
      axis1Opts,
      axis2Opts,
      scale,
    }
  },
}
</script>