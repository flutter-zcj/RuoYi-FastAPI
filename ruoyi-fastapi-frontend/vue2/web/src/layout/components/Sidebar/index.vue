<template>
  <div
    :class="['sidebar-theme-wrapper', { 'has-logo': showLogo }, settings.sideTheme]"
    :style="{
      backgroundColor:
        settings.sideTheme === 'theme-dark'
          ? variables.menuBackground
          : variables.menuLightBackground,
    }">
    <logo v-if="showLogo" :collapse="isCollapse" />
    <el-scrollbar :class="settings.sideTheme" wrap-class="scrollbar-wrapper">
      <el-menu
        :default-active="activeMenu"
        :collapse="isCollapse"
        :background-color="
          settings.sideTheme === 'theme-dark'
            ? variables.menuBackground
            : variables.menuLightBackground
        "
        :text-color="
          settings.sideTheme === 'theme-dark' ? variables.menuColor : variables.menuLightColor
        "
        :unique-opened="true"
        :active-text-color="settings.theme"
        :collapse-transition="false"
        mode="vertical">
        <sidebar-item
          v-for="(route, index) in sidebarRouters"
          :key="route.path + index"
          :item="route"
          :base-path="route.path" />
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script>
/**
 * 组件名称：Sidebar
 * 功能说明：侧边栏容器组件，展示 Logo 与多级菜单
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

import { mapGetters, mapState } from 'vuex'
import Logo from './Logo'
import SidebarItem from './SidebarItem'
import variables from '@/assets/styles/variables.scss'

export default
{
  components:
  {
    SidebarItem,
    Logo
  },

  computed:
  {
    ...mapState(['settings']),
    ...mapGetters(['sidebarRouters', 'sidebar']),

    /**
     * 默认激活菜单
     * @returns {String} 激活菜单路径
     */
    activeMenu()
    {
      const route = this.$route
      const { meta, path } = route
      // if set path, the sidebar will highlight the path you set
      if (meta.activeMenu)
      {
        return meta.activeMenu
      }
      return path
    },

    /**
     * 是否显示 Logo
     * @returns {Boolean} 是否显示
     */
    showLogo()
    {
      return this.$store.state.settings.sidebarLogo
    },

    /**
     * 样式变量
     * @returns {Object} 样式变量
     */
    variables()
    {
      return variables
    },

    /**
     * 是否折叠
     * @returns {Boolean} 是否折叠
     */
    isCollapse()
    {
      return !this.sidebar.opened
    }
  }
}
</script>