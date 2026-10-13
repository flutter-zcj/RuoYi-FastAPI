<template>
  <component :is="type" v-bind="linkProps(to)">
    <slot />
  </component>
</template>

<script>
/**
 * 组件名称：Sidebar
 * 功能说明：侧边栏链接组件，根据目标地址渲染内部路由或外部链接
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String|Object} to - 跳转目标
 *
 * 变量说明：
 *
 * 事件说明：
 * @event linkProps - 生成链接属性
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { isExternal } from '@/utils/validate'

export default
{
  props:
  {
    to: { type: [String, Object], required: true }
  },

  computed:
  {
    /**
     * 是否为外部链接
     * @returns {Boolean} 是否外部链接
     */
    isExternal()
    {
      return isExternal(this.to)
    },

    /**
     * 组件类型
     * @returns {String} 组件类型
     */
    type()
    {
      if (this.isExternal)
      {
        return 'a'
      }
      return 'router-link'
    }
  },

  methods:
  {
    /**
     * 生成链接属性
     * @param {String|Object} to - 跳转目标
     * @returns {Object} 链接属性
     */
    linkProps(to)
    {
      if (this.isExternal)
      {
        return {
          href: to,
          target: '_blank',
          rel: 'noopener',
        }
      }
      return {
        to: to,
      }
    }
  }
}
</script>