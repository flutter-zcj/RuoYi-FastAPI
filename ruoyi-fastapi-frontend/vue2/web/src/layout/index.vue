<style lang="scss" scoped>
@import '~@/assets/styles/mixin.scss';
@import '~@/assets/styles/variables.scss';
.app-wrapper { @include clearfix; position: relative; height: 100%; width: 100%;
  &.mobile.openSidebar { position: fixed; top: 0; }
}
.main-container:has(.fixed-header) { overflow: hidden; height: 100vh; }
.drawer-bg { position: absolute; top: 0; z-index: 999; width: 100%; height: 100%; background: #000; opacity: 0.3; }
.fixed-header { position: fixed; top: 0; right: 0; z-index: 9; width: calc(100% - #{$base-sidebar-width}); transition: width 0.28s; }
.hideSidebar .fixed-header { width: calc(100% - 54px); }
.sidebarHide .fixed-header { width: 100%; }
.mobile .fixed-header { width: 100%; }
</style>

<template>
  <div
    class="app-wrapper"
    :class="classObj"
    :style="{
      '--current-color': theme,
      '--current-color-light': theme + '1a',
      '--current-color-dark-bg': theme + '33',
    }">
    <div
      class="drawer-bg"
      v-if="device === 'mobile' && sidebar.opened"
      @click="handleClickOutside" />
    <sidebar class="sidebar-container" v-if="!sidebar.hide" />
    <div
      class="main-container"
      :class="{ hasTagsView: needTagsView, sidebarHide: sidebar.hide }">
      <div :class="{ 'fixed-header': fixedHeader }">
        <navbar @setLayout="setLayout" />
        <tags-view v-if="needTagsView" />
      </div>
      <app-main />
      <settings ref="settingRef" />
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：layout
 * 功能说明：整体布局组件，负责侧边栏、顶栏、标签页与主内容区组合
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event handleClickOutside - 点击遮罩关闭侧边栏
 * @event setLayout          - 打开设置面板
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { AppMain, Navbar, Settings, Sidebar, TagsView } from './components'
import ResizeMixin from './mixin/ResizeHandler'
import { mapState } from 'vuex'
import variables from '@/assets/styles/variables.scss'

export default
{
  name: 'Layout',

  mixins: [ResizeMixin],

  components:
  {
    AppMain,
    Navbar,
    Settings,
    Sidebar,
    TagsView,
  },

  computed:
  {
    ...mapState({
      theme: (state) => state.settings.theme,
      sideTheme: (state) => state.settings.sideTheme,
      sidebar: (state) => state.app.sidebar,
      device: (state) => state.app.device,
      needTagsView: (state) => state.settings.tagsView,
      fixedHeader: (state) => state.settings.fixedHeader,
    }),

    /**
     * 布局样式类
     * @returns {Object} 布局样式类对象
     */
    classObj()
    {
      return {
        hideSidebar: !this.sidebar.opened,
        openSidebar: this.sidebar.opened,
        withoutAnimation: this.sidebar.withoutAnimation,
        mobile: this.device === 'mobile',
      }
    },

    /**
     * 样式变量
     * @returns {Object} 样式变量
     */
    variables()
    {
      return variables
    }
  },

  methods:
  {
    /**
     * 点击遮罩关闭侧边栏
     */
    handleClickOutside()
    {
      this.$store.dispatch('app/closeSideBar', { withoutAnimation: false })
    },

    /**
     * 打开设置面板
     */
    setLayout()
    {
      this.$refs.settingRef.openSetting()
    }
  }
}
</script>