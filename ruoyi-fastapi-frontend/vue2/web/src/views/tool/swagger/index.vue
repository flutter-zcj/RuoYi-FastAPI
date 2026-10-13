<style lang="scss" scoped></style>

<template>
  <i-frame :src="url" />
</template>

<script>
/**
 * 组件名称：Swagger
 * 功能说明：接口文档页面，通过内嵌 iframe 展示后端接口文档
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String} url - 接口文档地址
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
  name: 'Swagger',

  components:
  {
    iFrame
  },

  data()
  {
    return {
      url: process.env.VUE_APP_BASE_API + '/proxy-docs',
    }
  }
}
</script>