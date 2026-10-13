<template>
  <div v-if="!item.hidden">
    <template
      v-if="
        hasOneShowingChild(item.children, item) &&
        (!onlyOneChild.children || onlyOneChild.noShowingChildren) &&
        !item.alwaysShow
      ">
      <app-link v-if="onlyOneChild.meta" :to="resolvePath(onlyOneChild.path, onlyOneChild.query)">
        <el-menu-item :class="{ 'submenu-title-noDropdown': !isNest }" :index="resolvePath(onlyOneChild.path)">
          <item :icon="onlyOneChild.meta.icon || (item.meta && item.meta.icon)" :title="onlyOneChild.meta.title" />
        </el-menu-item>
      </app-link>
    </template>

    <el-submenu
      v-else
      ref="subMenu"
      :index="resolvePath(item.path)"
      popper-append-to-body>
      <template slot="title">
        <item
          v-if="item.meta"
          :icon="item.meta && item.meta.icon"
          :title="item.meta.title" />
      </template>
      <sidebar-item
        class="nest-menu"
        v-for="(child, index) in item.children"
        :key="child.path + index"
        :is-nest="true"
        :item="child"
        :base-path="resolvePath(child.path)" />
    </el-submenu>
  </div>
</template>

<script>
/**
 * 组件名称：Sidebar
 * 功能说明：侧边栏菜单项组件，递归渲染单级或多级菜单
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Object}  item     - 路由对象
 * @prop {Boolean} isNest   - 是否为嵌套菜单
 * @prop {String}  basePath - 基础路径
 *
 * 变量说明：
 *
 * 事件说明：
 * @event hasOneShowingChild - 判断是否只有一个显示的子菜单
 * @event resolvePath        - 解析路由路径
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import path from 'path'
import { isExternal } from '@/utils/validate'
import Item from './Item'
import AppLink from './Link'
import FixiOSBug from './FixiOSBug'

export default
{
  name: 'SidebarItem',

  mixins: [FixiOSBug],

  components:
  {
    Item,
    AppLink
  },

  props:
  {
    // route object
    item: { type: Object, required: true },
    isNest: { type: Boolean, default: false },
    basePath: { type: String, default: '' }
  },

  data()
  {
    this.onlyOneChild = null
    return {}
  },

  methods:
  {
    /**
     * 判断是否只有一个显示的子菜单
     * @param {Array} children - 子菜单列表
     * @param {Object} parent - 父菜单对象
     * @returns {Boolean} 是否只有一个显示的子菜单
     */
    hasOneShowingChild(children = [], parent)
    {
      if (!children)
      {
        children = []
      }
      const showingChildren = children.filter((item) =>
      {
        if (item.hidden)
        {
          return false
        }
        // Temp set(will be used if only has one showing child)
        this.onlyOneChild = item
        return true
      })

      // When there is only one child router, the child router is displayed by default
      if (showingChildren.length === 1)
      {
        return true
      }

      // Show parent if there are no child router to display
      if (showingChildren.length === 0)
      {
        this.onlyOneChild = { ...parent, path: '', noShowingChildren: true }
        return true
      }

      return false
    },

    /**
     * 解析路由路径
     * @param {String} routePath - 路由路径
     * @param {String} routeQuery - 路由查询参数
     * @returns {String|Object} 解析后的路径
     */
    resolvePath(routePath, routeQuery)
    {
      if (isExternal(routePath))
      {
        return routePath
      }
      if (isExternal(this.basePath))
      {
        return this.basePath
      }
      if (routeQuery)
      {
        let query = JSON.parse(routeQuery)
        return { path: path.resolve(this.basePath, routePath), query: query }
      }
      return path.resolve(this.basePath, routePath)
    }
  }
}
</script>