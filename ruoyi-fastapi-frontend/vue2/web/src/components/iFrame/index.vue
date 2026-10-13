<template>
  <div v-loading="loading" :style="'height:' + height">
    <iframe
      :src="src"
      frameborder="no"
      style="width: 100%; height: 100%"
      scrolling="auto"/>
  </div>
</template>

<script>
/**
 * 组件名称：iFrame
 * 功能说明：iframe 内嵌页面组件，动态计算高度并显示加载状态
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} src - 内嵌页面地址
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
  props:
  {
    src: { type: String, required: true }
  },

  data()
  {
    return {
      height: document.documentElement.clientHeight - 94.5 + 'px;',
      loading: true,
      url: this.src
    }
  },

  mounted()
  {
    setTimeout(() =>
    {
      this.loading = false
    }, 300)
    const that = this
    window.onresize = function temp()
    {
      that.height = document.documentElement.clientHeight - 94.5 + 'px;'
    }
  }
}
</script>
