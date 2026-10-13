<style lang="scss" scoped>
.setting-drawer-content {
  .setting-drawer-title { margin-bottom: 12px; color: rgba(0, 0, 0, 0.85); font-size: 14px; line-height: 22px; font-weight: bold; }
  .setting-drawer-block-checbox {
    display: flex; justify-content: flex-start; align-items: center; margin-top: 10px; margin-bottom: 20px;
    .setting-drawer-block-checbox-item {
      position: relative; margin-right: 16px; border-radius: 2px; cursor: pointer;
      img { width: 48px; height: 48px; }
      .setting-drawer-block-checbox-selectIcon { position: absolute; top: 0; right: 0; width: 100%; height: 100%; padding-top: 15px; padding-left: 24px; color: #1890ff; font-weight: 700; font-size: 14px; }
    }
  }
}
.drawer-container {
  padding: 20px; font-size: 14px; line-height: 1.5; word-wrap: break-word;
  .drawer-title { margin-bottom: 12px; color: rgba(0, 0, 0, 0.85); font-size: 14px; line-height: 22px; }
  .drawer-item { color: rgba(0, 0, 0, 0.65); font-size: 14px; padding: 12px 0; }
  .drawer-switch { float: right; }
}
.nav-wrap {
  display: flex; justify-content: flex-start; align-items: center; margin-top: 10px; margin-bottom: 20px;
  .activeItem { border: 2px solid #{'var(--theme)'} !important; }
  .item { position: relative; margin-right: 16px; cursor: pointer; width: 56px; height: 48px; border-radius: 4px; background: #f0f2f5; border: 2px solid transparent; }
  .left {
    b:first-child { display: block; height: 30%; background: #fff; }
    b:last-child { width: 30%; background: #1b2a47; position: absolute; height: 100%; top: 0; border-radius: 4px 0 0 4px; }
  }
  .mix {
    b:first-child { border-radius: 4px 4px 0 0; display: block; height: 30%; background: #1b2a47; }
    b:last-child { width: 30%; background: #1b2a47; position: absolute; height: 70%; border-radius: 0 0 0 4px; }
  }
  .top {
    b:first-child { display: block; height: 30%; background: #1b2a47; border-radius: 4px 4px 0 0; }
  }
}
</style>

<template>
  <el-drawer
    size="280px"
    :visible="showSettings"
    :with-header="false"
    :append-to-body="true"
    :before-close="closeSetting"
    :lock-scroll="false">
    <div class="drawer-container">
      <div>
        <div class="setting-drawer-content">
          <div class="setting-drawer-title">
            <h3 class="drawer-title">菜单导航设置</h3>
          </div>
          <div class="nav-wrap">
            <el-tooltip content="左侧菜单" placement="bottom">
              <div
                class="item left"
                :class="{ activeItem: navType == 1 }"
                :style="{ '--theme': theme }"
                @click="handleNavType(1)">
                <b></b><b></b>
              </div>
            </el-tooltip>

            <el-tooltip content="混合菜单" placement="bottom">
              <div
                class="item mix"
                :class="{ activeItem: navType == 2 }"
                :style="{ '--theme': theme }"
                @click="handleNavType(2)">
                <b></b><b></b>
              </div>
            </el-tooltip>
            <el-tooltip content="顶部菜单" placement="bottom">
              <div
                class="item top"
                :class="{ activeItem: navType == 3 }"
                :style="{ '--theme': theme }"
                @click="handleNavType(3)">
                <b></b><b></b>
              </div>
            </el-tooltip>
          </div>
          <div class="setting-drawer-title">
            <h3 class="drawer-title">主题风格设置</h3>
          </div>
          <div class="setting-drawer-block-checbox">
            <div class="setting-drawer-block-checbox-item" @click="handleTheme('theme-dark')">
              <img src="@/assets/images/dark.svg" alt="dark" />
              <div
                class="setting-drawer-block-checbox-selectIcon"
                v-if="sideTheme === 'theme-dark'"
                style="display: block">
                <i class="anticon anticon-check" aria-label="图标: check">
                  <svg
                    class=""
                    viewBox="64 64 896 896"
                    data-icon="check"
                    width="1em"
                    height="1em"
                    :fill="theme"
                    aria-hidden="true"
                    focusable="false">
                    <path d="M912 190h-69.9c-9.8 0-19.1 4.5-25.1 12.2L404.7 724.5 207 474a32 32 0 0 0-25.1-12.2H112c-6.7 0-10.4 7.7-6.3 12.9l273.9 347c12.8 16.2 37.4 16.2 50.3 0l488.4-618.9c4.1-5.1.4-12.8-6.3-12.8z"/>
                  </svg>
                </i>
              </div>
            </div>
            <div class="setting-drawer-block-checbox-item" @click="handleTheme('theme-light')">
              <img src="@/assets/images/light.svg" alt="light" />
              <div
                class="setting-drawer-block-checbox-selectIcon"
                v-if="sideTheme === 'theme-light'"
                style="display: block">
                <i class="anticon anticon-check" aria-label="图标: check">
                  <svg
                    class=""
                    viewBox="64 64 896 896"
                    data-icon="check"
                    width="1em"
                    height="1em"
                    :fill="theme"
                    aria-hidden="true"
                    focusable="false">
                    <path d="M912 190h-69.9c-9.8 0-19.1 4.5-25.1 12.2L404.7 724.5 207 474a32 32 0 0 0-25.1-12.2H112c-6.7 0-10.4 7.7-6.3 12.9l273.9 347c12.8 16.2 37.4 16.2 50.3 0l488.4-618.9c4.1-5.1.4-12.8-6.3-12.8z"/>
                  </svg>
                </i>
              </div>
            </div>
          </div>

          <div class="drawer-item">
            <span>主题颜色</span>
            <theme-picker style="float: right; height: 26px; margin: -3px 8px 0 0" @change="themeChange" />
          </div>
        </div>

        <el-divider />

        <h3 class="drawer-title">系统布局配置</h3>

        <div class="drawer-item">
          <span>开启页签</span>
          <el-switch class="drawer-switch" v-model="tagsView" />
        </div>

        <div class="drawer-item">
          <span>持久化标签页</span>
          <el-switch
            class="drawer-switch"
            v-model="tagsViewPersist"
            :disabled="!tagsView"/>
        </div>

        <div class="drawer-item">
          <span>显示页签图标</span>
          <el-switch
            class="drawer-switch"
            v-model="tagsIcon"
            :disabled="!tagsView"/>
        </div>

        <div class="drawer-item">
          <span>标签页样式</span>
          <el-radio-group
            class="drawer-switch"
            v-model="tagsViewStyle"
            :disabled="!tagsView"
            size="mini">
            <el-radio-button label="card">卡片</el-radio-button>
            <el-radio-button label="chrome">谷歌</el-radio-button>
          </el-radio-group>
        </div>

        <div class="drawer-item">
          <span>固定 Header</span>
          <el-switch class="drawer-switch" v-model="fixedHeader" />
        </div>

        <div class="drawer-item">
          <span>显示 Logo</span>
          <el-switch class="drawer-switch" v-model="sidebarLogo" />
        </div>

        <div class="drawer-item">
          <span>动态标题</span>
          <el-switch class="drawer-switch" v-model="dynamicTitle" />
        </div>

        <div class="drawer-item">
          <span>底部版权</span>
          <el-switch class="drawer-switch" v-model="footerVisible" />
        </div>

        <el-divider />

        <el-button
          size="small"
          type="primary"
          plain
          icon="el-icon-document-add"
          @click="saveSetting">保存配置</el-button>
        <el-button
          size="small"
          plain
          icon="el-icon-refresh"
          @click="resetSetting">重置配置</el-button>
      </div>
    </div>
  </el-drawer>
</template>

<script>
/**
 * 组件名称：Settings
 * 功能说明：系统布局设置抽屉，支持菜单导航、主题风格与系统布局配置
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

import ThemePicker from '@/components/ThemePicker'

export default
{
  components: { ThemePicker },

  expose: ['openSetting'],

  data()
  {
    return {
      theme: this.$store.state.settings.theme,
      sideTheme: this.$store.state.settings.sideTheme,
      navType: this.$store.state.settings.navType,
      showSettings: false,
    }
  },

  computed:
  {
    fixedHeader:
    {
      /**
       * 读取固定 Header 配置
       * @returns {Boolean} 是否固定 Header
       */
      get()
      {
        return this.$store.state.settings.fixedHeader
      },

      /**
       * 更新固定 Header 配置
       * @param {Boolean} val - 是否固定 Header
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'fixedHeader',
          value: val,
        })
      },
    },

    tagsViewPersist:
    {
      /**
       * 读取标签页持久化配置
       * @returns {Boolean} 是否持久化标签页
       */
      get()
      {
        return this.$store.state.settings.tagsViewPersist
      },

      /**
       * 更新标签页持久化配置
       * @param {Boolean} val - 是否持久化标签页
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'tagsViewPersist',
          value: val,
        })
      },
    },

    tagsView:
    {
      /**
       * 读取开启页签配置
       * @returns {Boolean} 是否开启页签
       */
      get()
      {
        return this.$store.state.settings.tagsView
      },

      /**
       * 更新开启页签配置
       * @param {Boolean} val - 是否开启页签
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'tagsView',
          value: val,
        })
      },
    },

    tagsIcon:
    {
      /**
       * 读取显示页签图标配置
       * @returns {Boolean} 是否显示页签图标
       */
      get()
      {
        return this.$store.state.settings.tagsIcon
      },

      /**
       * 更新显示页签图标配置
       * @param {Boolean} val - 是否显示页签图标
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'tagsIcon',
          value: val,
        })
      },
    },

    tagsViewStyle:
    {
      /**
       * 读取标签页样式配置
       * @returns {String} 标签页样式
       */
      get()
      {
        return this.$store.state.settings.tagsViewStyle
      },

      /**
       * 更新标签页样式配置
       * @param {String} val - 标签页样式
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'tagsViewStyle',
          value: val,
        })
      },
    },

    sidebarLogo:
    {
      /**
       * 读取显示 Logo 配置
       * @returns {Boolean} 是否显示 Logo
       */
      get()
      {
        return this.$store.state.settings.sidebarLogo
      },

      /**
       * 更新显示 Logo 配置
       * @param {Boolean} val - 是否显示 Logo
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'sidebarLogo',
          value: val,
        })
      },
    },

    dynamicTitle:
    {
      /**
       * 读取动态标题配置
       * @returns {Boolean} 是否开启动态标题
       */
      get()
      {
        return this.$store.state.settings.dynamicTitle
      },

      /**
       * 更新动态标题配置并同步页面标题
       * @param {Boolean} val - 是否开启动态标题
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'dynamicTitle',
          value: val,
        })
        this.$store.dispatch('settings/setTitle', this.$store.state.settings.title)
      },
    },

    footerVisible:
    {
      /**
       * 读取底部版权配置
       * @returns {Boolean} 是否显示底部版权
       */
      get()
      {
        return this.$store.state.settings.footerVisible
      },

      /**
       * 更新底部版权配置
       * @param {Boolean} val - 是否显示底部版权
       */
      set(val)
      {
        this.$store.dispatch('settings/changeSetting', {
          key: 'footerVisible',
          value: val,
        })
      },
    },
  },

  watch:
  {
    navType:
    {
      /**
       * 监听导航模式变化并同步侧边栏与路由
       * @param {Number} val - 当前导航模式
       */
      handler(val)
      {
        if (val == 1) {
          this.$store.dispatch('app/toggleSideBarHide', false)
        }
        if (val == 2) {
        }
        if (val == 3) {
          this.$store.dispatch('app/toggleSideBarHide', true)
        }
        if ([1, 3].includes(val)) {
          this.$store.commit('SET_SIDEBAR_ROUTERS', this.$store.state.permission.defaultRoutes)
        }
      },
      immediate: true,
      deep: true,
    },
  },

  methods:
  {
    /**
     * 更新主题颜色
     * @param {String} val - 主题颜色值
     */
    themeChange(val)
    {
      this.$store.dispatch('settings/changeSetting', {
        key: 'theme',
        value: val,
      })
      this.theme = val
    },

    /**
     * 更新侧边栏主题风格
     * @param {String} val - 侧边栏主题值
     */
    handleTheme(val)
    {
      this.$store.dispatch('settings/changeSetting', {
        key: 'sideTheme',
        value: val,
      })
      this.sideTheme = val
    },

    /**
     * 更新导航模式
     * @param {Number} val - 导航模式
     */
    handleNavType(val)
    {
      this.$store.dispatch('settings/changeSetting', {
        key: 'navType',
        value: val,
      })
      this.navType = val
    },

    /**
     * 打开设置抽屉
     */
    openSetting()
    {
      this.showSettings = true
    },

    /**
     * 关闭设置抽屉
     */
    closeSetting()
    {
      this.showSettings = false
    },

    /**
     * 保存布局配置到本地缓存
     */
    saveSetting()
    {
      this.$modal.loading('正在保存到本地，请稍候...')
      if (!this.tagsViewPersist) {
        this.$cache.local.remove('tags-view-visited')
      }
      this.$cache.local.set(
        'layout-setting',
        `{
            "navType":${this.navType},
            "tagsView":${this.tagsView},
            "tagsIcon":${this.tagsIcon},
            "tagsViewStyle":"${this.tagsViewStyle}",
            "tagsViewPersist":${this.tagsViewPersist},
            "fixedHeader":${this.fixedHeader},
            "sidebarLogo":${this.sidebarLogo},
            "dynamicTitle":${this.dynamicTitle},
            "footerVisible":${this.footerVisible},
            "sideTheme":"${this.sideTheme}",
            "theme":"${this.theme}"
          }`
      )
      setTimeout(this.$modal.closeLoading(), 1000)
    },

    /**
     * 重置布局配置并刷新页面
     */
    resetSetting()
    {
      this.$modal.loading('正在清除设置缓存并刷新，请稍候...')
      this.$cache.local.remove('tags-view-visited')
      this.$cache.local.remove('layout-setting')
      setTimeout('window.location.reload()', 1000)
    },
  },
}
</script>