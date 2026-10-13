<style lang="scss" scoped>
.navbar.nav3 {
  .hamburger-container { display: none !important; }
}
.navbar {
  height: 50px; overflow: hidden; position: relative; background: #fff; box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08); display: flex; align-items: center; box-sizing: border-box;
  .hamburger-container {
    line-height: 46px; height: 100%; cursor: pointer; transition: background 0.3s; -webkit-tap-highlight-color: transparent; display: flex; align-items: center; flex-shrink: 0; margin-right: 8px;
    &:hover { background: rgba(0, 0, 0, 0.025); }
  }
  .breadcrumb-container { flex-shrink: 0; }
  .topmenu-container { position: absolute; left: 50px; }
  .topbar-container { flex: 1; min-width: 0; display: flex; align-items: center; overflow: hidden; margin-left: 8px; }
  .right-menu {
    height: 100%; line-height: 50px; display: flex; align-items: center; margin-left: auto;
    &:focus { outline: none; }
    .right-menu-item {
      display: inline-block; padding: 0 8px; height: 100%; font-size: 18px; color: #5a5e66; vertical-align: text-bottom;
      &.hover-effect {
        cursor: pointer; transition: background 0.3s;
        &:hover { background: rgba(0, 0, 0, 0.025); }
      }
    }
    .avatar-container {
      margin-right: 0px; padding-right: 0px;
      .avatar-wrapper {
        right: 8px; position: relative;
        .user-avatar { cursor: pointer; width: 30px; height: 30px; border-radius: 50%; vertical-align: middle; border-style: none; }
        .user-nickname { position: relative; left: 2px; font-size: 14px; font-weight: bold; }
        .el-icon-caret-bottom { cursor: pointer; position: absolute; right: -20px; top: 25px; font-size: 12px; }
      }
    }
  }
}
</style>

<template>
  <div class="navbar" :class="'nav' + navType">
    <hamburger
      class="hamburger-container"
      id="hamburger-container"
      :is-active="sidebar.opened"
      @toggleClick="toggleSideBar"/>

    <breadcrumb
      class="breadcrumb-container"
      v-if="navType == 1"
      id="breadcrumb-container"/>
    <top-nav
      class="topmenu-container"
      v-if="navType == 2"
      id="topmenu-container"/>
    <template v-if="navType == 3">
      <logo v-show="showLogo" :collapse="false"></logo>
      <top-bar class="topbar-container" id="topbar-container"/>
    </template>
    <div class="right-menu">
      <template v-if="device !== 'mobile'">
        <search class="right-menu-item" id="header-search" />

        <el-tooltip
          content="源码地址"
          effect="dark"
          placement="bottom">
          <ruo-yi-git class="right-menu-item hover-effect" id="ruoyi-git"/>
        </el-tooltip>

        <el-tooltip
          content="文档地址"
          effect="dark"
          placement="bottom">
          <ruo-yi-doc class="right-menu-item hover-effect" id="ruoyi-doc"/>
        </el-tooltip>

        <screenfull class="right-menu-item hover-effect" id="screenfull" />

        <el-tooltip
          content="布局大小"
          effect="dark"
          placement="bottom">
          <size-select class="right-menu-item hover-effect" id="size-select"/>
        </el-tooltip>

        <el-tooltip
          content="消息通知"
          effect="dark"
          placement="bottom">
          <header-notice class="right-menu-item hover-effect" id="header-notice"/>
        </el-tooltip>
      </template>

      <el-dropdown class="avatar-container right-menu-item hover-effect" trigger="hover">
        <div class="avatar-wrapper">
          <img class="user-avatar" :src="avatar" />
          <span class="user-nickname"> {{ nickName }} </span>
        </div>
        <el-dropdown-menu slot="dropdown">
          <router-link to="/user/profile">
            <el-dropdown-item>个人中心</el-dropdown-item>
          </router-link>
          <el-dropdown-item v-if="setting" @click.native="setLayout">
            <span>布局设置</span>
          </el-dropdown-item>
          <el-dropdown-item @click.native="lockScreen">
            <span>锁定屏幕</span>
          </el-dropdown-item>
          <el-dropdown-item divided @click.native="logout">
            <span>退出登录</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：Navbar
 * 功能说明：顶部导航栏组件，集成面包屑、顶部菜单、搜索、全屏、布局设置、消息通知与用户操作
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event setLayout - 打开布局设置抽屉
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { mapGetters } from 'vuex'
import Breadcrumb from '@/components/Breadcrumb'
import TopNav from './TopNav'
import TopBar from './TopBar'
import Logo from './Sidebar/Logo'
import Hamburger from '@/components/Hamburger'
import Screenfull from '@/components/Screenfull'
import SizeSelect from '@/components/SizeSelect'
import Search from '@/components/HeaderSearch'
import RuoYiGit from '@/components/RuoYi/Git'
import RuoYiDoc from '@/components/RuoYi/Doc'
import HeaderNotice from './HeaderNotice'

export default
{
  components:
  {
    Breadcrumb,
    Logo,
    TopNav,
    TopBar,
    Hamburger,
    Screenfull,
    SizeSelect,
    Search,
    RuoYiGit,
    RuoYiDoc,
    HeaderNotice,
  },

  computed:
  {
    ...mapGetters(['sidebar', 'avatar', 'device', 'nickName']),

    setting:
    {
      /**
       * 读取布局设置抽屉显示状态
       * @returns {Boolean} 是否显示设置抽屉
       */
      get()
      {
        return this.$store.state.settings.showSettings
      },
    },

    navType:
    {
      /**
       * 读取导航模式
       * @returns {Number} 导航模式
       */
      get()
      {
        return this.$store.state.settings.navType
      },
    },

    showLogo:
    {
      /**
       * 读取是否显示 Logo
       * @returns {Boolean} 是否显示 Logo
       */
      get()
      {
        return this.$store.state.settings.sidebarLogo
      },
    },
  },

  methods:
  {
    /**
     * 切换侧边栏展开/折叠状态
     */
    toggleSideBar()
    {
      this.$store.dispatch('app/toggleSideBar')
    },

    /**
     * 触发打开布局设置事件
     * @param {Event} event - 点击事件对象
     */
    setLayout(event)
    {
      this.$emit('setLayout')
    },

    /**
     * 锁定屏幕并跳转到锁屏页
     */
    lockScreen()
    {
      const currentPath = this.$route.fullPath
      this.$store.dispatch('lock/lockScreen', currentPath).then(() => {
        this.$router.push('/lock')
      })
    },

    /**
     * 二次确认后注销并退出系统
     */
    logout()
    {
      this.$confirm('确定注销并退出系统吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          this.$store.dispatch('LogOut').then(() => {
            location.href = '/index'
          })
        })
        .catch(() => {})
    },
  },
}
</script>