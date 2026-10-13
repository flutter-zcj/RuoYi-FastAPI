<style lang="scss" scoped>
.svg-icon { overflow: hidden; width: 1em; height: 1em; vertical-align: -0.15em; fill: currentColor; }
.svg-external-icon { display: inline-block; background-color: currentColor; mask-size: cover !important; }
</style>

<template>
  <div
    class="svg-external-icon svg-icon"
    v-if="isExternal"
    :style="styleExternalIcon"
    v-on="$listeners"/>
  <svg
    :class="svgClass"
    v-else
    aria-hidden="true"
    v-on="$listeners">
    <use :xlink:href="iconName"/>
  </svg>
</template>

<script>
/**
 * 组件名称：SvgIcon
 * 功能说明：svg 图标组件，支持外链图标与本地 symbol 图标
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} iconClass - 图标名称
 * @prop {String} className - 自定义类名
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

import { isExternal } from '@/utils/validate'

export default
{
  name: 'SvgIcon',

  props:
  {
    iconClass: { type: String, required: true },
    className: { type: String, default: '' }
  },

  computed:
  {
    /**
     * 判断图标是否为外链
     * @returns {Boolean} 是否外链
     */
    isExternal()
    {
      return isExternal(this.iconClass)
    },

    /**
     * 生成图标名称
     * @returns {String} 图标名称
     */
    iconName()
    {
      return `#icon-${this.iconClass}`
    },

    /**
     * 计算图标的 class 名称
     * @returns {String} class 名称
     */
    svgClass()
    {
      if (this.className)
      {
        return 'svg-icon ' + this.className
      } else
      {
        return 'svg-icon'
      }
    },

    /**
     * 计算外链图标的样式
     * @returns {Object} 样式对象
     */
    styleExternalIcon()
    {
      return {
        mask: `url(${this.iconClass}) no-repeat 50% 50%`,
        '-webkit-mask': `url(${this.iconClass}) no-repeat 50% 50%`
      }
    }
  }
}
</script>
