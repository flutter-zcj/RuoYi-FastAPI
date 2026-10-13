<style lang="scss" scoped>
.screenfull-svg { display: inline-block; width: 20px; height: 20px; fill: #5a5e66; vertical-align: 10px; cursor: pointer; }
</style>

<template>
  <div>
    <svg-icon :icon-class="isFullscreen ? 'exit-fullscreen' : 'fullscreen'" @click="click"/>
  </div>
</template>

<script>
/**
 * 组件名称：Screenfull
 * 功能说明：全屏切换组件，支持浏览器全屏与退出全屏
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

import screenfull from 'screenfull'

export default
{
  name: 'Screenfull',

  data()
  {
    return {
      isFullscreen: false
    }
  },

  mounted()
  {
    this.init()
  },

  beforeDestroy()
  {
    this.destroy()
  },

  methods:
  {
    /**
     * 切换全屏状态
     * @returns {Boolean|undefined} 浏览器不支持全屏时返回 false
     */
    click()
    {
      if (!screenfull.isEnabled)
      {
        this.$message({ message: '你的浏览器不支持全屏', type: 'warning' })
        return false
      }
      screenfull.toggle()
    },

    /**
     * 同步全屏状态
     */
    change()
    {
      this.isFullscreen = screenfull.isFullscreen
    },

    /**
     * 初始化全屏状态监听
     */
    init()
    {
      if (screenfull.isEnabled)
      {
        screenfull.on('change', this.change)
      }
    },

    /**
     * 销毁全屏状态监听
     */
    destroy()
    {
      if (screenfull.isEnabled)
      {
        screenfull.off('change', this.change)
      }
    }
  }
}
</script>
