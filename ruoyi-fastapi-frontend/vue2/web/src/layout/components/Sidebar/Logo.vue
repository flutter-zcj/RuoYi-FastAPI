<style lang="scss" scoped>
.sidebarLogoFade-enter-active { transition: opacity 1.5s; }
.sidebarLogoFade-enter, .sidebarLogoFade-leave-to { opacity: 0; }
.sidebar-logo-container { position: relative; overflow: hidden; height: 50px; line-height: 50px; background: #2b2f3a; text-align: center;
  & .sidebar-logo-link { height: 100%; width: 100%;
    & .sidebar-logo { width: 32px; height: 32px; margin-right: 12px; vertical-align: middle; }
    & .sidebar-title { display: inline-block; margin: 0; color: #fff; font-weight: 600; font-size: 14px; font-family: Avenir, Helvetica Neue, Arial, Helvetica, sans-serif; line-height: 50px; vertical-align: middle; }
  }
  &.collapse { .sidebar-logo { margin-right: 0px; } }
}
</style>

<template>
  <div
    class="sidebar-logo-container"
    :class="{ collapse: collapse }"
    :style="{
      backgroundColor:
        sideTheme === 'theme-dark' && navType !== 3
          ? variables.menuBackground
          : variables.menuLightBackground,
    }">
    <transition name="sidebarLogoFade">
      <router-link
        class="sidebar-logo-link"
        v-if="collapse"
        key="collapse"
        to="/">
        <img
          class="sidebar-logo"
          v-if="logo"
          :src="logo" />
        <h1
          class="sidebar-title"
          v-else
          :style="{
            color:
              sideTheme === 'theme-dark' && navType !== 3
                ? variables.logoTitleColor
                : variables.logoLightTitleColor,
          }">
          {{ title }}
        </h1>
      </router-link>
      <router-link
        class="sidebar-logo-link"
        v-else
        key="expand"
        to="/">
        <img
          class="sidebar-logo"
          v-if="logo"
          :src="logo" />
        <h1
          class="sidebar-title"
          :style="{
            color:
              sideTheme === 'theme-dark' && navType !== 3
                ? variables.logoTitleColor
                : variables.logoLightTitleColor,
          }">
          {{ title }}
        </h1>
      </router-link>
    </transition>
  </div>
</template>

<script>
/**
 * 组件名称：Sidebar
 * 功能说明：侧边栏 Logo 组件，展示系统 Logo 与标题
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} collapse - 是否折叠侧边栏
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

import logoImg from '@/assets/logo/logo.png'
import variables from '@/assets/styles/variables.scss'

export default
{
  name: 'SidebarLogo',

  props:
  {
    collapse: { type: Boolean, required: true }
  },

  data()
  {
    return {
      title: process.env.VUE_APP_TITLE,
      logo: logoImg,
    }
  },

  computed:
  {
    /**
     * 样式变量
     * @returns {Object} 样式变量
     */
    variables()
    {
      return variables
    },

    /**
     * 侧边栏主题
     * @returns {String} 主题名称
     */
    sideTheme()
    {
      return this.$store.state.settings.sideTheme
    },

    /**
     * 导航模式
     * @returns {Number} 导航模式
     */
    navType()
    {
      return this.$store.state.settings.navType
    }
  }
}
</script>