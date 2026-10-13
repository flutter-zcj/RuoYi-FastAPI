<style lang="scss" scoped>
.app-main {
  min-height: calc(100vh - 50px); width: 100%; position: relative; overflow: hidden;
  &:fullscreen, &:-webkit-full-screen, &:-moz-full-screen, &:-ms-fullscreen { background: #fff; overflow-y: auto; }
}
.fixed-header + .app-main { overflow-y: auto; scrollbar-gutter: auto; height: calc(100vh - 50px); min-height: 0px; }
.app-main:has(.copyright) { padding-bottom: 36px; }
.fixed-header + .app-main { margin-top: 50px; }
.hasTagsView {
  .app-main { min-height: calc(100vh - 84px); }
  .fixed-header + .app-main { margin-top: 84px; height: calc(100vh - 84px); min-height: 0px; }
}
@media screen and (max-width: 991px) {
  .fixed-header + .app-main { padding-bottom: max(60px, calc(constant(safe-area-inset-bottom) + 40px)); padding-bottom: max(60px, calc(env(safe-area-inset-bottom) + 40px)); overscroll-behavior-y: none; }
  .hasTagsView .fixed-header + .app-main { padding-bottom: max(60px, calc(constant(safe-area-inset-bottom) + 40px)); padding-bottom: max(60px, calc(env(safe-area-inset-bottom) + 40px)); overscroll-behavior-y: none; }
}
@supports (-webkit-touch-callout: none) {
  @media screen and (max-width: 991px) {
    .fixed-header + .app-main { padding-bottom: max(17px, calc(constant(safe-area-inset-bottom) + 10px)); padding-bottom: max(17px, calc(env(safe-area-inset-bottom) + 10px)); height: calc(100svh - 50px); height: calc(100dvh - 50px); }
    .hasTagsView .fixed-header + .app-main { padding-bottom: max(17px, calc(constant(safe-area-inset-bottom) + 10px)); padding-bottom: max(17px, calc(env(safe-area-inset-bottom) + 10px)); height: calc(100svh - 84px); height: calc(100dvh - 84px); }
  }
}
</style>

<style lang="scss">
::-webkit-scrollbar { width: 6px; height: 6px; }
::-webkit-scrollbar-track { background-color: #f1f1f1; }
::-webkit-scrollbar-thumb { background-color: #c0c0c0; border-radius: 3px; }
</style>

<template>
  <section class="app-main">
    <transition name="fade-transform" mode="out-in">
      <keep-alive :include="cachedViews">
        <router-view v-if="!$route.meta.link" :key="key" />
      </keep-alive>
    </transition>
    <iframe-toggle />
    <copyright />
  </section>
</template>

<script>
/**
 * 组件名称：AppMain
 * 功能说明：主内容区组件，承载路由视图、iframe 切换与底部版权
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
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import copyright from './Copyright/index'
import iframeToggle from './IframeToggle/index'

export default
{
  name: 'AppMain',

  components: { iframeToggle, copyright },

  computed:
  {
    /**
     * 获取已缓存的页面视图列表
     * @returns {Array} 缓存视图列表
     */
    cachedViews()
    {
      return this.$store.state.tagsView.cachedViews
    },

    /**
     * 获取当前路由路径作为视图 key
     * @returns {String} 当前路由路径
     */
    key()
    {
      return this.$route.path
    },
  },

  watch:
  {
    $route()
    {
      this.addIframe()
    },
  },

  mounted()
  {
    this.addIframe()
  },

  methods:
  {
    /**
     * 将带 link 的路由添加为 iframe 视图
     */
    addIframe()
    {
      const { name } = this.$route
      if (name && this.$route.meta.link) {
        this.$store.dispatch('tagsView/addIframeView', this.$route)
      }
    },
  },
}
</script>