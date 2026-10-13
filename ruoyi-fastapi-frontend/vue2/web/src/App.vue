<style lang="scss" scoped>
#app .theme-picker { display: none; }
</style>

<template>
  <div id="app">
    <router-view/>
    <theme-picker/>
  </div>
</template>

<script>
/**
 * 组件名称：App
 * 功能说明：应用根组件，承载路由视图与主题选择器
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event refreshTimezoneOnVisible - 页面可见时刷新设备时区
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { refreshDeviceTimezone } from '@/utils/time'
import ThemePicker from '@/components/ThemePicker'

export default
{
  name: 'App',

  components: { ThemePicker },

  mounted()
  {
    window.addEventListener('focus', refreshDeviceTimezone)
    document.addEventListener('visibilitychange', this.refreshTimezoneOnVisible)
  },

  beforeDestroy()
  {
    window.removeEventListener('focus', refreshDeviceTimezone)
    document.removeEventListener('visibilitychange', this.refreshTimezoneOnVisible)
  },

  methods:
  {
    /**
     * 页面重新可见时刷新设备时区
     */
    refreshTimezoneOnVisible()
    {
      if (document.visibilityState === 'visible') refreshDeviceTimezone()
    },
  },
}
</script>