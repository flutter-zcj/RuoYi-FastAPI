<style lang="scss" scoped></style>

<template>
  <transition-group name="fade-transform" mode="out-in">
    <inner-link
      v-for="(item, index) in iframeViews"
      :key="item.path"
      :iframeId="'iframe' + index"
      v-show="$route.path === item.path"
      :src="iframeUrl(item.meta.link, item.query)"></inner-link>
  </transition-group>
</template>

<script>
/**
 * 组件名称：IframeToggle
 * 功能说明：iframe 页面切换组件，按标签页缓存并渲染内嵌页面
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
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import InnerLink from '../InnerLink/index'

export default
{
  components: { InnerLink },

  computed:
  {
    /**
     * 获取已缓存的 iframe 视图列表
     * @returns {Array} iframe 视图列表
     */
    iframeViews()
    {
      return this.$store.state.tagsView.iframeViews
    },
  },

  methods:
  {
    /**
     * 拼接 iframe 地址与查询参数
     * @param {String} url - 基础地址
     * @param {Object} query - 查询参数对象
     * @returns {String} 拼接后的完整地址
     */
    iframeUrl(url, query)
    {
      if (Object.keys(query).length > 0) {
        let params = Object.keys(query)
          .map((key) => key + '=' + query[key])
          .join('&')
        return url + '?' + params
      }
      return url
    },
  },
}
</script>