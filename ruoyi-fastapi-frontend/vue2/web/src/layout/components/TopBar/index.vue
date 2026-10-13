<style lang="scss">
.topbar-menu.el-menu--horizontal .el-submenu__title, .topbar-menu.el-menu--horizontal .el-menu-item { padding: 0 10px !important; }
.el-menu--horizontal .el-menu--popup .el-menu-item:hover { background-color: #f5f7fa !important; }
.topbar-menu.el-menu--horizontal > .el-submenu .el-submenu__title { float: left; height: 47px !important; margin: 0 15px !important; line-height: 50px !important; color: #303133; }
.topbar-menu .el-submenu .el-submenu__icon-arrow { position: static; margin-top: 0px; margin-left: 8px; vertical-align: middle; }
.topbar-menu.el-menu--horizontal .el-submenu__title, .topbar-menu.el-menu--horizontal .el-menu-item { height: 55px; }
.el-menu--horizontal .el-menu .el-menu-item, .el-menu--horizontal .el-menu .el-submenu__title { color: #303133; }
</style>

<template>
  <el-menu
    class="topbar-menu"
    :default-active="activeMenu"
    :active-text-color="theme"
    mode="horizontal">
    <sidebar-item
      :key="route.path + index"
      v-for="(route, index) in topMenus"
      :item="route"
      :base-path="route.path" />

    <el-submenu
      class="el-submenu__hide-arrow"
      index="more"
      v-if="moreRoutes.length > 0">
      <template slot="title">更多菜单</template>
      <sidebar-item
        :key="route.path + index"
        v-for="(route, index) in moreRoutes"
        :item="route"
        :base-path="route.path" />
    </el-submenu>
  </el-menu>
</template>

<script>
/**
 * 组件名称：TopBar
 * 功能说明：顶部栏菜单组件，展示一级菜单并支持超出数量折叠
 * 研发时间：2025/12/19
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event setVisibleNumber - 根据宽度计算设置显示栏数
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import SidebarItem from '../Sidebar/SidebarItem'

export default
{
  components:
  {
    SidebarItem
  },

  data()
  {
    return {
      // 顶部栏初始数
      visibleNumber: 5,
    }
  },

  computed:
  {
    /**
     * 主题色
     * @returns {String} 主题色值
     */
    theme()
    {
      return this.$store.state.settings.theme
    },

    /**
     * 顶部显示菜单
     * @returns {Array} 顶部菜单集合
     */
    topMenus()
    {
      return this.$store.state.permission.sidebarRouters
        .filter((f) => !f.hidden)
        .slice(0, this.visibleNumber)
    },

    /**
     * 折叠菜单
     * @returns {Array} 折叠菜单集合
     */
    moreRoutes()
    {
      const sidebarRouters = this.$store.state.permission.sidebarRouters
      return sidebarRouters
        .filter((f) => !f.hidden)
        .slice(this.visibleNumber, sidebarRouters.length - this.visibleNumber)
    },

    // 默认激活的菜单
    /**
     * 默认激活的菜单
     * @returns {String} 激活菜单路径
     */
    activeMenu()
    {
      const { meta, path } = this.$route
      if (meta.activeMenu)
      {
        return meta.activeMenu
      }
      return path
    }
  },

  beforeMount()
  {
    window.addEventListener('resize', this.setVisibleNumber)
  },

  mounted()
  {
    this.setVisibleNumber()
  },

  beforeDestroy()
  {
    window.removeEventListener('resize', this.setVisibleNumber)
  },

  methods:
  {
    // 根据宽度计算设置显示栏数
    /**
     * 根据宽度计算设置显示栏数
     */
    setVisibleNumber()
    {
      const width = document.body.getBoundingClientRect().width / 3
      this.visibleNumber = parseInt(width / 85)
    }
  }
}
</script>