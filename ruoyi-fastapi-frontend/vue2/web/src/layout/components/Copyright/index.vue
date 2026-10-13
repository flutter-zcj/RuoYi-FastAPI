<style lang="scss" scoped>
.copyright { position: fixed; bottom: 0; left: 0; right: 0; height: 36px; padding: 10px 20px; text-align: right; background-color: #f8f8f8; color: #666; font-size: 14px; border-top: 1px solid #e7e7e7; z-index: 999; }
</style>

<template>
  <footer class="copyright" v-if="visible">
    <span>{{ content }}</span>
  </footer>
</template>

<script>
/**
 * 组件名称：Copyright
 * 功能说明：底部版权信息组件，按配置读取并展示版权内容
 * 研发时间：2025/08/29
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
 * 2025/08/29     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  computed:
  {
    /**
     * 读取底部版权显示状态
     * @returns {Boolean} 是否显示底部版权
     */
    visible()
    {
      return this.$store.state.settings.footerVisible
    },

    /**
     * 读取底部版权内容
     * @returns {String} 版权内容
     */
    content()
    {
      return this.$store.state.settings.footerContent
    },
  },
}
</script>