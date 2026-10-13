<style lang="scss" scoped></style>

<template>
  <!-- <i-frame :src="url" /> -->
  <div>
    <div>我是数据监控</div>
  </div>
</template>

<script>
/**
 * 组件名称：Druid
 * 功能说明：数据监控页面
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String} url - Druid 监控地址
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import iFrame from '@/components/iFrame/index'

export default
{
  name: 'Druid',

  components:
  {
    iFrame
  },

  /**
   * 组件数据
   * @returns {Object} 组件响应式数据
   */
  data()
  {
    return {
      url: process.env.VUE_APP_BASE_API + '/druid/login.html',
    }
  },
}
</script>