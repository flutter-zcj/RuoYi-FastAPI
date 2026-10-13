<style lang="scss" scoped></style>

<template>
  <div
    :style="'height:' + height"
    v-loading="loading"
    element-loading-text="正在加载页面，请稍候！">
    <iframe
      :id="iframeId"
      style="width: 100%; height: 100%"
      :src="src"
      frameborder="no"></iframe>
  </div>
</template>

<script>
/**
 * 组件名称：InnerLink
 * 功能说明：内嵌 iframe 页面组件，根据高度与加载状态渲染外部链接
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} src      - iframe 地址
 * @prop {String} iframeId - iframe 元素 id
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

export default
{
  props:
  {
    src: {
      type: String,
      default: '/',
    },
    iframeId: {
      type: String,
    },
  },

  data()
  {
    return {
      loading: false,
      height: document.documentElement.clientHeight - 94.5 + 'px;',
    }
  },

  mounted()
  {
    var _this = this
    const iframeId = ('#' + this.iframeId).replace(/\//g, '\\/')
    const iframe = document.querySelector(iframeId)
    // iframe页面loading控制
    if (iframe.attachEvent) {
      this.loading = true
      iframe.attachEvent('onload', function () {
        _this.loading = false
      })
    } else {
      this.loading = true
      iframe.onload = function () {
        _this.loading = false
      }
    }
  },
}
</script>