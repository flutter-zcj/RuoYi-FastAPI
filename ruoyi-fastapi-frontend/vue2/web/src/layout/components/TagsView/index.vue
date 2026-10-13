<style lang="scss" scoped>
$tags-bar-height: 34px;
.tags-view-container { display: flex; align-items: center; width: 100%; height: $tags-bar-height; border-bottom: 1px solid #d8dce5; background: #fff; overflow: hidden;
  $btn-width: 28px;
  $btn-color: #71717a;
  $btn-hover-bg: #f0f2f5;
  $btn-hover-color: #303133;
  $btn-disabled-color: #c0c4cc;
  $divider: 1px solid #d8dce5;
  .tags-nav-btn { display: flex; align-items: center; justify-content: center; flex-shrink: 0; width: $btn-width; height: $tags-bar-height; cursor: pointer; color: $btn-color; font-size: 13px; user-select: none; transition: background 0.15s, color 0.15s;
    &:hover:not(.disabled) { background: $btn-hover-bg; color: $btn-hover-color; }
    &.disabled { color: $btn-disabled-color; cursor: not-allowed; }
    &--left { border-right: $divider; }
    &--right { border-left: $divider; }
  }
  .tags-view-wrapper { flex: 1; min-width: 0; height: 100%;
    .tags-view-item { display: inline-block; position: relative; height: 26px; margin-left: 5px; padding: 0 8px; border: 1px solid #d8dce5; border-radius: 3px; background: #fff; font-size: 12px; line-height: 26px; color: #495060; cursor: pointer;
      &:first-of-type { margin-left: 6px; }
      &:last-of-type { margin-right: 15px; }
    }
  }
  &:not(.tags-view-container--chrome) .tags-view-wrapper .tags-view-item.active { border-color: #42b983; background-color: #42b983; color: #fff;
    &::before { content: ''; position: relative; display: inline-block; width: 8px; height: 8px; margin-right: 2px; border-radius: 50%; background: #fff; }
  }
  &:not(.tags-view-container--chrome) .tags-view-wrapper .tags-view-item.active.has-icon::before { content: none !important; }
  .tags-action-dropdown { display: flex; align-items: center; flex-shrink: 0; }
  .tags-action-btn { display: flex; align-items: center; justify-content: center; width: $btn-width; height: $tags-bar-height; border-left: $divider; cursor: pointer; color: $btn-color; font-size: 13px; user-select: none; transition: background 0.15s, color 0.15s;
    &:hover { background: $btn-hover-bg; color: $btn-hover-color; }
  }
  .tags-refresh-btn { width: 60px; }
  .contextmenu { position: fixed; z-index: 3000; margin: 0; padding: 5px 0; border-radius: 4px; background: #fff; font-size: 12px; font-weight: 400; color: #333; box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, 0.3); list-style-type: none;
    li { margin: 0; padding: 7px 16px; cursor: pointer;
      &:hover { background: #eee; }
    }
  }
  &.tags-view-container--chrome { --chrome-strip-bg: #ffffff; --chrome-strip-border: #e4e7ed; --chrome-tab-text: #606266; overflow: visible; background: var(--chrome-strip-bg); border-bottom: 1px solid var(--chrome-strip-border); align-items: flex-end;
    .tags-nav-btn { align-self: stretch; height: auto; min-height: $tags-bar-height; border-color: var(--chrome-strip-border); }
    .tags-action-btn { border-color: var(--chrome-strip-border); }
    .tags-view-wrapper {
      .tags-view-item { display: inline-flex !important; position: relative; z-index: 1; align-items: center; justify-content: center; height: 30px; min-height: 30px; margin: 0 0 -1px; padding: 0 12px; border: none !important; border-radius: 0; background: transparent !important; font-size: 13px; font-weight: 400; line-height: 1.2; color: var(--chrome-tab-text) !important; padding-top: 0 !important; box-shadow: none !important; transition: background 0.12s ease, color 0.12s ease, border-radius 0.12s ease;
        &::before, &::after { content: '' !important; position: absolute; bottom: 0; display: block !important; width: var(--chrome-wing-r); height: var(--chrome-wing-r); margin: 0 !important; border-radius: 0 !important; background: transparent !important; pointer-events: none; transition: box-shadow 0.12s ease; }
        &::before { left: calc(-1 * var(--chrome-wing-r)); border-bottom-right-radius: var(--chrome-wing-r) !important; box-shadow: none; }
        &::after { right: calc(-1 * var(--chrome-wing-r)); border-bottom-left-radius: var(--chrome-wing-r) !important; box-shadow: none; }
        &:first-of-type { margin-left: 6px; }
        &:last-of-type { margin-right: 10px; }
        &:not(.active) + .tags-view-item:not(.active) { border-left: 1px solid #e4e7ed; padding-left: 11px; }
        &:hover:not(.active) { background: #f5f7fa !important; border-radius: 6px 6px 0 0; color: #303133 !important; }
        &.active { height: 31px; min-height: 31px; padding: 0 14px; border: none !important; border-radius: var(--chrome-wing-r) var(--chrome-wing-r) 0 0; background: var(--chrome-tab-active-bg) !important; font-weight: 500; color: var(--chrome-tab-text-active) !important; box-shadow: 0 1px 4px rgba(0, 0, 0, 0.06);
          &::before { box-shadow: calc(var(--chrome-wing-r) * 0.5) calc(var(--chrome-wing-r) * 0.5) 0 calc(var(--chrome-wing-r) * 0.5) var(--chrome-tab-active-bg); }
          &::after { box-shadow: calc(var(--chrome-wing-r) * -0.5) calc(var(--chrome-wing-r) * 0.5) 0 calc(var(--chrome-wing-r) * 0.5) var(--chrome-tab-active-bg); }
        }
        .el-icon-close { margin-left: 3px;
          &:before { vertical-align: -2px; }
        }
      }
    }
  }
}
</style>

<style lang="scss">
.tags-view-wrapper {
  .el-scrollbar { height: 100%; overflow: hidden; }
  .el-scrollbar__wrap { display: flex; align-items: center; height: 34px !important; overflow-x: auto; overflow-y: hidden; scrollbar-width: none; -ms-overflow-style: none;
    &::-webkit-scrollbar { width: 0; height: 0; }
    .tags-view-container:hover & {
      &::-webkit-scrollbar { width: 6px; height: 6px; }
      &::-webkit-scrollbar-track { background: transparent; }
      &::-webkit-scrollbar-thumb { background-color: rgba(0, 0, 0, 0.2); border-radius: 3px; transition: background-color 0.2s;
        &:hover { background-color: rgba(0, 0, 0, 0.4); }
      }
    }
  }
  .el-scrollbar__bar { z-index: 10; opacity: 0; transition: opacity 0.3s;
    .tags-view-container:hover & { opacity: 1; }
  }
  .tags-view-container--chrome & {
    .el-scrollbar { position: relative; }
    .el-scrollbar__wrap { scrollbar-width: none; -ms-overflow-style: none;
      &::-webkit-scrollbar { width: 0 !important; height: 0 !important; }
    }
    .el-scrollbar__bar.is-horizontal { z-index: 20; height: 6px !important; }
  }
  .tags-view-item {
    .el-icon-close { width: 16px; height: 16px; border-radius: 50%; text-align: center; vertical-align: 2px; transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1); transform-origin: 100% 50%;
      &:before { display: inline-block; transform: scale(0.6); vertical-align: -3px; }
      &:hover { background-color: #b4bccc; color: #fff; }
    }
  }
}
.main-container.fullscreen-mode { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; overflow: hidden; margin-left: 0 !important; transition: none !important; }
.main-container.fullscreen-mode .fixed-header { display: block !important; position: fixed; top: 0; left: 0; right: 0; z-index: 1000; width: 100% !important; transition: none !important; }
.main-container.fullscreen-mode .fixed-header .navbar { display: none !important; }
.main-container.fullscreen-mode .app-main { position: fixed; top: 34px; left: 0; right: 0; bottom: 0; margin: 0 !important; padding: 0 !important; height: calc(100vh - 34px) !important; min-height: calc(100vh - 34px) !important; overflow: auto; }
</style>

<template>
  <div
    id="tags-view-container"
    class="tags-view-container"
    :class="{ 'tags-view-container--chrome': tagsViewStyle === 'chrome' }"
    :style="chromeVars">
    <!-- 左切换箭头 -->
    <span
      class="tags-nav-btn tags-nav-btn--left"
      :class="{ disabled: !canScrollLeft }"
      @click="scrollLeft">
      <i class="el-icon-arrow-left" />
    </span>

    <!-- 标签滚动区 -->
    <scroll-pane
      ref="scrollPane"
      class="tags-view-wrapper"
      @scroll="handleScroll"
      @updateArrows="updateArrowState">
      <router-link
        class="tags-view-item"
        :class="{ active: isActive(tag), 'has-icon': tagsIcon }"
        v-for="tag in visitedViews"
        ref="tag"
        :key="tag.path"
        :to="{ path: tag.path, query: tag.query, fullPath: tag.fullPath }"
        tag="span"
        :style="tagActiveStyle(tag)"
        @click.middle.native="!isAffix(tag) ? closeSelectedTag(tag) : ''"
        @contextmenu.prevent.native="openMenu(tag, $event)">
        <svg-icon
          v-if="tagsIcon && tag.meta && tag.meta.icon && tag.meta.icon !== '#'"
          :icon-class="tag.meta.icon"
          style="margin-right: 3px" />
        {{ tag.title }}
        <span
          class="el-icon-close"
          v-if="!isAffix(tag)"
          @click.prevent.stop="closeSelectedTag(tag)" />
      </router-link>
    </scroll-pane>

    <!-- 右切换箭头 -->
    <span
      class="tags-nav-btn tags-nav-btn--right"
      :class="{ disabled: !canScrollRight }"
      @click="scrollRight">
      <i class="el-icon-arrow-right" />
    </span>

    <!-- 下拉操作菜单 -->
    <el-dropdown
      class="tags-action-dropdown"
      trigger="click"
      placement="bottom-end"
      @command="handleDropdownCommand">
      <span class="tags-action-btn">
        <i class="el-icon-arrow-down" />
      </span>
      <el-dropdown-menu class="tags-dropdown-menu" slot="dropdown">
        <el-dropdown-item
          v-if="!isAffix(selectedDropdownTag)"
          command="close"
          icon="el-icon-close">关闭当前</el-dropdown-item>
        <el-dropdown-item
          command="closeOthers"
          icon="el-icon-circle-close">关闭其他</el-dropdown-item>
        <el-dropdown-item
          command="closeLeft"
          :disabled="isFirstView()"
          icon="el-icon-back">关闭左侧</el-dropdown-item>
        <el-dropdown-item
          command="closeRight"
          :disabled="isLastView()"
          icon="el-icon-right">关闭右侧</el-dropdown-item>
        <el-dropdown-item
          command="closeAll"
          icon="el-icon-circle-close">全部关闭</el-dropdown-item>
        <el-dropdown-item command="fullscreen" divided>
          <template v-if="!isFullscreen"><i class="el-icon-full-screen"></i>全屏显示</template>
          <template v-else><i class="el-icon-close"></i>退出全屏</template>
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>

    <!-- 刷新按钮 -->
    <span
      class="tags-action-btn tags-refresh-btn"
      title="刷新页面"
      @click="refreshSelectedTag(selectedDropdownTag)">
      <i class="el-icon-refresh-right" /> 刷新
    </span>

    <!-- 右键上下文菜单 -->
    <ul
      class="contextmenu"
      v-show="visible"
      :style="{ left: left + 'px', top: top + 'px' }">
      <li @click="refreshSelectedTag(selectedTag)">
        <i class="el-icon-refresh-right"></i> 刷新页面
      </li>
      <li v-if="!isAffix(selectedTag)" @click="closeSelectedTag(selectedTag)">
        <i class="el-icon-close"></i> 关闭当前
      </li>
      <li @click="closeOthersTags"><i class="el-icon-circle-close"></i> 关闭其他</li>
      <li v-if="!isFirstView()" @click="closeLeftTags">
        <i class="el-icon-back"></i> 关闭左侧
      </li>
      <li v-if="!isLastView()" @click="closeRightTags">
        <i class="el-icon-right"></i> 关闭右侧
      </li>
      <li @click="closeAllTags(selectedTag)"><i class="el-icon-circle-close"></i> 全部关闭</li>
    </ul>
  </div>
</template>

<script>
/**
 * 组件名称：TagsView
 * 功能说明：标签导航组件，支持标签切换、关闭、刷新与全屏显示
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event handleKeyDown         - 键盘按键事件处理
 * @event mixHexWithWhite       - 按比例混合十六进制颜色与白色
 * @event isActive              - 判断标签是否处于激活状态
 * @event tagActiveStyle        - 计算标签激活样式
 * @event isAffix               - 判断标签是否为固定标签
 * @event isFirstView           - 判断是否为第一个标签
 * @event isLastView            - 判断是否为最后一个标签
 * @event filterAffixTags       - 过滤需要固定的标签
 * @event initTags              - 初始化标签
 * @event addTags               - 添加标签
 * @event moveToCurrentTag      - 滚动到当前标签
 * @event scrollLeft            - 向左滚动
 * @event scrollRight           - 向右滚动
 * @event updateArrowState      - 更新左右箭头状态
 * @event toggleFullscreen      - 切换全屏模式
 * @event handleDropdownCommand - 处理下拉菜单命令
 * @event refreshSelectedTag    - 刷新选中标签
 * @event closeSelectedTag      - 关闭选中标签
 * @event closeRightTags        - 关闭右侧标签
 * @event closeLeftTags         - 关闭左侧标签
 * @event closeOthersTags       - 关闭其他标签
 * @event closeAllTags          - 关闭全部标签
 * @event toLastView            - 跳转到最后一个标签
 * @event openMenu              - 打开右键菜单
 * @event closeMenu             - 关闭右键菜单
 * @event handleScroll          - 处理滚动
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import ScrollPane from './ScrollPane'
import path from 'path'

export default
{
  components:
  {
    ScrollPane
  },

  data()
  {
    return {
      visible: false,
      top: 0,
      left: 0,
      selectedTag: {},
      affixTags: [],
      canScrollLeft: false,
      canScrollRight: false,
      isFullscreen: false,
      hiddenElements: [],
    }
  },

  computed:
  {
    /**
     * 已访问的标签列表
     * @returns {Array} 已访问的标签列表
     */
    visitedViews()
    {
      return this.$store.state.tagsView.visitedViews
    },

    /**
     * 所有路由
     * @returns {Array} 路由集合
     */
    routes()
    {
      return this.$store.state.permission.routes
    },

    /**
     * 主题色
     * @returns {String} 主题色值
     */
    theme()
    {
      return this.$store.state.settings.theme
    },

    /**
     * 是否显示标签图标
     * @returns {Boolean} 是否显示标签图标
     */
    tagsIcon()
    {
      return this.$store.state.settings.tagsIcon
    },

    /**
     * 标签页样式
     * @returns {String} 标签页样式名称
     */
    tagsViewStyle()
    {
      return this.$store.state.settings.tagsViewStyle
    },

    /**
     * 下拉菜单选中的标签
     * @returns {Object} 选中的标签对象
     */
    selectedDropdownTag()
    {
      return this.visitedViews.find((v) => this.isActive(v)) || {}
    },

    /**
     * chrome 主题样式变量
     * @returns {Object} 样式变量对象
     */
    chromeVars()
    {
      if (this.tagsViewStyle !== 'chrome') return {}
      const primary = this.theme || '#409EFF'
      return {
        '--chrome-tab-active-bg': this.mixHexWithWhite(primary, 0.15),
        '--chrome-tab-text-active': primary,
        '--chrome-wing-r': '14px',
      }
    }
  },

  watch:
  {
    /**
     * 监听路由变化，添加并滚动到当前标签
     */
    $route()
    {
      this.addTags()
      this.moveToCurrentTag()
    },

    /**
     * 监听右键菜单显隐，绑定或解绑点击事件
     * @param {Boolean} value - 右键菜单是否可见
     */
    visible(value)
    {
      if (value)
      {
        document.body.addEventListener('click', this.closeMenu)
      }
      else
      {
        document.body.removeEventListener('click', this.closeMenu)
      }
    },

    /**
     * 监听标签变化，更新箭头状态
     */
    visitedViews()
    {
      this.$nextTick(() =>
      {
        this.updateArrowState()
      })
    }
  },

  mounted()
  {
    this.initTags()
    this.addTags()
    window.addEventListener('resize', this.updateArrowState)
    window.addEventListener('keydown', this.handleKeyDown)
  },

  beforeDestroy()
  {
    window.removeEventListener('resize', this.updateArrowState)
    window.removeEventListener('keydown', this.handleKeyDown)
  },

  methods:
  {
    /**
     * 键盘按键事件处理
     * @param {Object} event - 键盘事件对象
     */
    handleKeyDown(event)
    {
      // 当按下Esc键且处于全屏状态时，退出全屏
      if (event.key === 'Escape' && this.isFullscreen)
      {
        this.toggleFullscreen()
      }
    },

    /**
     * 按比例混合十六进制颜色与白色
     * @param {String} hex - 十六进制颜色值
     * @param {Number} ratio - 混合比例
     * @returns {String} 混合后的颜色
     */
    mixHexWithWhite(hex, ratio)
    {
      const clean = hex.replace('#', '')
      const r = parseInt(clean.substring(0, 2), 16)
      const g = parseInt(clean.substring(2, 4), 16)
      const b = parseInt(clean.substring(4, 6), 16)
      const mr = Math.round(r * ratio + 255 * (1 - ratio))
      const mg = Math.round(g * ratio + 255 * (1 - ratio))
      const mb = Math.round(b * ratio + 255 * (1 - ratio))
      return `rgb(${mr}, ${mg}, ${mb})`
    },

    /**
     * 判断标签是否处于激活状态
     * @param {Object} route - 标签路由对象
     * @returns {Boolean} 是否处于激活状态
     */
    isActive(route)
    {
      return route.path === this.$route.path
    },

    /**
     * 计算标签激活样式
     * @param {Object} tag - 标签对象
     * @returns {Object} 激活样式对象
     */
    tagActiveStyle(tag)
    {
      if (!this.isActive(tag) || this.tagsViewStyle !== 'card') return {}
      return {
        'background-color': this.theme,
        'border-color': this.theme,
      }
    },

    /**
     * 判断标签是否为固定标签
     * @param {Object} tag - 标签对象
     * @returns {Boolean} 是否为固定标签
     */
    isAffix(tag)
    {
      return tag && tag.meta && tag.meta.affix
    },

    /**
     * 判断是否为第一个标签
     * @returns {Boolean} 是否为第一个标签
     */
    isFirstView()
    {
      try
      {
        const tag =
          this.selectedTag && this.selectedTag.fullPath
            ? this.selectedTag
            : this.selectedDropdownTag
        return tag.fullPath === '/index' || tag.fullPath === this.visitedViews[1].fullPath
      }
      catch (err)
      {
        return false
      }
    },

    /**
     * 判断是否为最后一个标签
     * @returns {Boolean} 是否为最后一个标签
     */
    isLastView()
    {
      try
      {
        const tag =
          this.selectedTag && this.selectedTag.fullPath
            ? this.selectedTag
            : this.selectedDropdownTag
        return tag.fullPath === this.visitedViews[this.visitedViews.length - 1].fullPath
      }
      catch (err)
      {
        return false
      }
    },

    /**
     * 过滤需要固定的标签
     * @param {Array} routes - 路由集合
     * @param {String} basePath - 基础路径
     * @returns {Array} 固定标签集合
     */
    filterAffixTags(routes, basePath = '/')
    {
      let tags = []
      routes.forEach((route) =>
      {
        if (route.meta && route.meta.affix)
        {
          const tagPath = path.resolve(basePath, route.path)
          tags.push({
            fullPath: tagPath,
            path: tagPath,
            name: route.name,
            meta: { ...route.meta },
          })
        }
        if (route.children)
        {
          const tempTags = this.filterAffixTags(route.children, route.path)
          if (tempTags.length >= 1)
          {
            tags = [...tags, ...tempTags]
          }
        }
      })
      return tags
    },

    /**
     * 初始化标签
     */
    initTags()
    {
      if (this.$store.state.settings.tagsViewPersist)
      {
        this.$store.dispatch('tagsView/loadPersistedViews')
      }
      const affixTags = (this.affixTags = this.filterAffixTags(this.routes))
      for (const tag of affixTags)
      {
        if (tag.name)
        {
          this.$store.dispatch('tagsView/addAffixView', tag)
        }
      }
    },

    /**
     * 添加标签
     */
    addTags()
    {
      const { name } = this.$route
      if (name)
      {
        this.$store.dispatch('tagsView/addView', this.$route)
      }
    },

    /**
     * 滚动到当前标签
     */
    moveToCurrentTag()
    {
      const tags = this.$refs.tag
      this.$nextTick(() =>
      {
        for (const tag of tags)
        {
          if (tag.to.path === this.$route.path)
          {
            this.$refs.scrollPane.moveToTarget(tag)
            if (tag.to.fullPath !== this.$route.fullPath)
            {
              this.$store.dispatch('tagsView/updateVisitedView', this.$route)
            }
            break
          }
        }
      })
    },

    /**
     * 向左滚动
     */
    scrollLeft()
    {
      if (!this.canScrollLeft) return
      this.$refs.scrollPane.scrollToStart()
    },

    /**
     * 向右滚动
     */
    scrollRight()
    {
      if (!this.canScrollRight) return
      this.$refs.scrollPane.scrollToEnd()
    },

    /**
     * 更新左右箭头状态
     */
    updateArrowState()
    {
      this.$nextTick(() =>
      {
        if (this.$refs.scrollPane)
        {
          const state = this.$refs.scrollPane.getScrollState()
          this.canScrollLeft = state.canLeft
          this.canScrollRight = state.canRight
        }
      })
    },

    /**
     * 切换全屏模式
     */
    toggleFullscreen()
    {
      const mainContainer = document.querySelector('.main-container')
      const navbar = document.querySelector('.navbar')
      const sidebar = document.querySelector('.sidebar-container')
      if (!mainContainer) return

      if (!this.isFullscreen)
      {
        mainContainer.classList.add('fullscreen-mode')
        document.body.style.overflow = 'hidden'
        const elementsToHide = [
          { el: navbar, originalDisplay: (navbar && navbar.style.display) || '' },
          { el: sidebar, originalDisplay: (sidebar && sidebar.style.display) || '' },
        ]
        elementsToHide.forEach((item) =>
        {
          if (item.el && item.el.style.display !== 'none')
          {
            item.originalDisplay = item.el.style.display
            item.el.style.display = 'none'
            this.hiddenElements.push(item)
          }
        })
        this.isFullscreen = true
      }
      else
      {
        mainContainer.classList.remove('fullscreen-mode')
        document.body.style.overflow = ''
        this.hiddenElements.forEach((item) =>
        {
          if (item.el)
          {
            item.el.style.display = item.originalDisplay
          }
        })
        this.hiddenElements = []
        const btn = document.querySelector('.tags-action-btn')
        if (btn) btn.blur()
        this.isFullscreen = false
      }
    },

    /**
     * 处理下拉菜单命令
     * @param {String} command - 下拉菜单命令
     */
    handleDropdownCommand(command)
    {
      const tag = this.selectedDropdownTag
      this.selectedTag = tag
      switch (command)
      {
        case 'refresh':
          this.refreshSelectedTag(tag)
          break
        case 'fullscreen':
          this.toggleFullscreen()
          break
        case 'close':
          this.closeSelectedTag(tag)
          break
        case 'closeOthers':
          this.closeOthersTags()
          break
        case 'closeLeft':
          this.closeLeftTags()
          break
        case 'closeRight':
          this.closeRightTags()
          break
        case 'closeAll':
          this.closeAllTags(tag)
          break
      }
    },

    /**
     * 刷新选中标签
     * @param {Object} view - 标签对象
     */
    refreshSelectedTag(view)
    {
      this.$tab.refreshPage(view)
      if (this.$route.meta.link)
      {
        this.$store.dispatch('tagsView/delIframeView', this.$route)
      }
    },

    /**
     * 关闭选中标签
     * @param {Object} view - 标签对象
     */
    closeSelectedTag(view)
    {
      this.$tab.closePage(view).then(({ visitedViews }) =>
      {
        if (this.isActive(view))
        {
          this.toLastView(visitedViews, view)
        }
      })
    },

    /**
     * 关闭右侧标签
     */
    closeRightTags()
    {
      this.$tab.closeRightPage(this.selectedTag).then((visitedViews) =>
      {
        if (!visitedViews.find((i) => i.fullPath === this.$route.fullPath))
        {
          this.toLastView(visitedViews)
        }
      })
    },

    /**
     * 关闭左侧标签
     */
    closeLeftTags()
    {
      this.$tab.closeLeftPage(this.selectedTag).then((visitedViews) =>
      {
        if (!visitedViews.find((i) => i.fullPath === this.$route.fullPath))
        {
          this.toLastView(visitedViews)
        }
      })
    },

    /**
     * 关闭其他标签
     */
    closeOthersTags()
    {
      this.$router.push(this.selectedTag.fullPath).catch(() => {})
      this.$tab.closeOtherPage(this.selectedTag).then(() =>
      {
        this.moveToCurrentTag()
      })
    },

    /**
     * 关闭全部标签
     * @param {Object} view - 标签对象
     */
    closeAllTags(view)
    {
      this.$tab.closeAllPage().then(({ visitedViews }) =>
      {
        if (this.affixTags.some((tag) => tag.path === this.$route.path))
        {
          return
        }
        this.toLastView(visitedViews, view)
      })
    },

    /**
     * 跳转到最后一个标签
     * @param {Array} visitedViews - 已访问标签列表
     * @param {Object} view - 标签对象
     */
    toLastView(visitedViews, view)
    {
      const latestView = visitedViews.slice(-1)[0]
      if (latestView)
      {
        this.$router.push(latestView.fullPath)
      }
      else
      {
        if (view && view.name === 'Dashboard')
        {
          this.$router.replace({ path: '/redirect' + view.fullPath })
        }
        else
        {
          this.$router.push('/')
        }
      }
    },

    /**
     * 打开右键菜单
     * @param {Object} tag - 标签对象
     * @param {Object} e - 鼠标事件对象
     */
    openMenu(tag, e)
    {
      this.left = e.clientX
      this.top = e.clientY
      this.visible = true
      this.selectedTag = tag
    },

    /**
     * 关闭右键菜单
     */
    closeMenu()
    {
      this.visible = false
    },

    /**
     * 处理滚动
     */
    handleScroll()
    {
      this.closeMenu()
      this.updateArrowState()
    }
  }
}
</script>