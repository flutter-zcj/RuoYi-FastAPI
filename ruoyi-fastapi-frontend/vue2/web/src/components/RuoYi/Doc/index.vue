<style lang="scss" scoped>
</style>

<template>
  <div>
    <svg-icon icon-class="question" @click="goto"/>
  </div>
</template>

<script>
/**
 * 组件名称：Doc
 * 功能说明：文档入口组件，点击后在新窗口打开 RuoYi 官方文档
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'RuoYiDoc',

  data()
  {
    return {
      url: 'http://doc.ruoyi.vip/ruoyi-vue'
    }
  },

  methods:
  {
    /**
     * 在新窗口打开文档地址
     */
    goto()
    {
      window.open(this.url)
    }
  }
}
</script>