<style lang="scss">
.topmenu-container.el-menu--horizontal > .el-menu-item { float: left; height: 50px !important; padding: 0 5px !important; margin: 0 10px !important; line-height: 50px !important; color: #303133 !important; }
.topmenu-container.el-menu--horizontal > .el-menu-item.is-active, .el-menu--horizontal > .el-submenu.is-active .el-submenu__title { border-bottom: 2px solid #{'var(--theme)'} !important; color: #303133; }
.topmenu-container.el-menu--horizontal > .el-submenu .el-submenu__title { float: left; height: 50px !important; padding: 0 5px !important; margin: 0 10px !important; line-height: 50px !important; color: #303133 !important; }
</style>

<template>
  <el-menu
    :default-active="activeMenu"
    mode="horizontal"
    @select="handleSelect">
    <template v-for="(item, index) in topMenus">
      <el-menu-item
        :style="{ '--theme': theme }"
        :index="item.path"
        :key="index"
        v-if="index < visibleNumber">
        <svg-icon v-if="item.meta && item.meta.icon && item.meta.icon !== '#'" :icon-class="item.meta.icon" />
        {{ item.meta.title }}
      </el-menu-item>
    </template>

    <!-- 顶部菜单超出数量折叠 -->
    <el-submenu
      :style="{ '--theme': theme }"
      index="more"
      :key="visibleNumber"
      v-if="topMenus.length > visibleNumber">
      <template slot="title">更多菜单</template>
      <template v-for="(item, index) in topMenus">
        <el-menu-item
          :index="item.path"
          :key="index"
          v-if="index >= visibleNumber">
          <svg-icon v-if="item.meta && item.meta.icon && item.meta.icon !== '#'" :icon-class="item.meta.icon" />
          {{ item.meta.title }}
        </el-menu-item>
      </template>
    </el-submenu>
  </el-menu>
</template>

<script>
/**
 * 组件名称：TopNav
 * 功能说明：顶部导航菜单组件，展示一级菜单并支持超出数量折叠
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event setVisibleNumber - 根据宽度计算设置显示栏数
 * @event handleSelect      - 菜单选择事件
 * @event activeRoutes      - 当前激活的路由
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { constantRoutes } from '@/router'
import { isHttp } from '@/utils/validate'

// 隐藏侧边栏路由
const hideList = ['/index', '/user/profile']

export default
{
  data()
  {
    return {
      // 顶部栏初始数
      visibleNumber: 5,
      // 当前激活菜单的 index
      currentIndex: undefined,
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

    // 顶部显示菜单
    /**
     * 顶部显示菜单
     * @returns {Array} 顶部菜单集合
     */
    topMenus()
    {
      let topMenus = []
      this.routers.map((menu) =>
      {
        if (menu.hidden !== true)
        {
          // 兼容顶部栏一级菜单内部跳转
          if (menu.path === '/' && menu.children)
          {
            topMenus.push(menu.children[0])
          }
          else
          {
            topMenus.push(menu)
          }
        }
      })
      return topMenus
    },

    // 所有的路由信息
    /**
     * 所有的路由信息
     * @returns {Array} 路由信息集合
     */
    routers()
    {
      return this.$store.state.permission.topbarRouters
    },

    // 设置子路由
    /**
     * 设置子路由
     * @returns {Array} 子路由集合
     */
    childrenMenus()
    {
      var childrenMenus = []
      this.routers.map((router) =>
      {
        for (var item in router.children)
        {
          if (router.children[item].parentPath === undefined)
          {
            if (router.path === '/')
            {
              router.children[item].path = '/' + router.children[item].path
            }
            else
            {
              if (!isHttp(router.children[item].path))
              {
                router.children[item].path = router.path + '/' + router.children[item].path
              }
            }
            router.children[item].parentPath = router.path
          }
          childrenMenus.push(router.children[item])
        }
      })
      return constantRoutes.concat(childrenMenus)
    },

    // 默认激活的菜单
    /**
     * 默认激活的菜单
     * @returns {String} 激活菜单路径
     */
    activeMenu()
    {
      const path = this.$route.path
      let activePath = path
      if (path !== undefined && path.lastIndexOf('/') > 0 && hideList.indexOf(path) === -1)
      {
        const tmpPath = path.substring(1, path.length)
        if (!this.$route.meta.link)
        {
          activePath = '/' + tmpPath.substring(0, tmpPath.indexOf('/'))
        }
      }
      else if (!this.$route.children)
      {
        activePath = path
        this.$store.dispatch('app/toggleSideBarHide', true)
      }
      this.activeRoutes(activePath)
      return activePath
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
    },

    // 菜单选择事件
    /**
     * 菜单选择事件
     * @param {String} key - 选中的菜单路径
     * @param {Array} keyPath - 选中菜单路径数组
     */
    handleSelect(key, keyPath)
    {
      this.currentIndex = key
      const route = this.routers.find((item) => item.path === key)
      if (isHttp(key))
      {
        // http(s):// 路径新窗口打开
        window.open(key, '_blank')
      }
      else if (!route || !route.children)
      {
        // 没有子路由路径内部打开
        const routeMenu = this.childrenMenus.find((item) => item.path === key)
        if (routeMenu && routeMenu.query)
        {
          let query = JSON.parse(routeMenu.query)
          this.$router.push({ path: key, query: query })
        }
        else
        {
          this.$router.push({ path: key })
        }
        this.$store.dispatch('app/toggleSideBarHide', true)
      }
      else
      {
        // 显示左侧联动菜单
        this.activeRoutes(key)
        this.$store.dispatch('app/toggleSideBarHide', false)
      }
    },

    // 当前激活的路由
    /**
     * 当前激活的路由
     * @param {String} key - 激活的菜单路径
     */
    activeRoutes(key)
    {
      var routes = []
      if (this.childrenMenus && this.childrenMenus.length > 0)
      {
        this.childrenMenus.map((item) =>
        {
          if (key == item.parentPath || (key == 'index' && '' == item.path))
          {
            routes.push(item)
          }
        })
      }
      if (routes.length > 0)
      {
        this.$store.commit('SET_SIDEBAR_ROUTERS', routes)
      }
      else
      {
        this.$store.dispatch('app/toggleSideBarHide', true)
      }
    }
  }
}
</script>