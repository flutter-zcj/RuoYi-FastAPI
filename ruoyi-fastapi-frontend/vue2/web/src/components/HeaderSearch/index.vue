<style lang="scss" scoped>
::v-deep { .el-dialog__header { padding: 6px !important; }
  .highlight { color: red; font-weight: 600; }
  .is-active .highlight { color: rgba(255, 255, 255, 0.9); font-weight: 600; }
}
.header-search { .search-icon { cursor: pointer; font-size: 18px; vertical-align: middle; } }
.result-count { padding: 6px 16px 0; font-size: 12px; color: #aaa;
  strong { color: red; font-weight: 600; }
}
.result-wrap { height: 280px; margin: 4px 0;
  .search-item { display: flex; align-items: center; padding-right: 10px; height: 48px; border-radius: 4px; transition: background 0.15s;
    .left { width: 60px; text-align: center; flex-shrink: 0;
      .menu-icon { width: 18px; height: 18px; }
    }
    .search-info { display: flex; flex-direction: column; justify-content: flex-start; width: 100%; padding-left: 5px; margin-top: 10px; flex: 1; overflow: hidden;
      .menu-title, .menu-path { height: 20px; overflow: hidden; white-space: nowrap; text-overflow: ellipsis; }
      .menu-path { color: #ccc; font-size: 10px; }
    }
  }
  .search-item:hover { cursor: pointer; }
  .empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%;
    .empty-icon { color: #e0e0e0; font-size: 42px; margin-bottom: 14px; }
    .empty-text { color: #999; font-size: 14px; margin: 0 0 6px;
      strong { color: #666; }
    }
    .empty-tip { color: #bbb; font-size: 12px; margin: 0; }
  }
}
.search-footer { display: flex; align-items: center; gap: 28px; padding: 10px 20px; border-top: 1px solid #f0f0f0; color: #999; font-size: 12px;
  .shortcut-item { display: flex; align-items: center; gap: 5px; }
  kbd { display: inline-flex; align-items: center; justify-content: center; min-width: 20px; height: 20px; padding: 0 5px; border: 1px solid #ddd; border-radius: 4px; background: #f7f7f7; color: #555; font-size: 11px; font-family: inherit; line-height: 1; box-shadow: 0 1px 0 #ccc; }
}
</style>

<template>
  <div class="header-search">
    <svg-icon
      class-name="search-icon"
      icon-class="search"
      @click.stop="click"/>
    <el-dialog
      :visible.sync="show"
      width="600px"
      :show-close="false"
      append-to-body
      @close="close"
      @opened="onDialogOpened">
      <el-input
        v-model="search"
        ref="headerSearchSelectRef"
        size="large"
        prefix-icon="el-icon-search"
        placeholder="菜单搜索，支持标题、URL模糊查询"
        clearable
        @input="querySearch"
        @keyup.enter.native="selectActiveResult"
        @keydown.up.native="navigateResult('up')"
        @keydown.down.native="navigateResult('down')">
      </el-input>

      <div
        class="result-count"
        v-if="search && options.length > 0">
        找到 <strong>{{ options.length }}</strong> 个结果
      </div>

      <el-scrollbar wrap-class="right-scrollbar-wrapper">
        <div class="result-wrap">
          <template v-if="options.length > 0">
            <div
              class="search-item"
              :class="{ 'is-active': index === activeIndex }"
              :key="item.path"
              :style="activeStyle(index)"
              v-for="(item, index) in options"
              @mouseenter="activeIndex = index"
              @mouseleave="activeIndex = -1">
              <div class="left">
                <svg-icon
                  class="menu-icon"
                  :icon-class="item.icon"/>
              </div>
              <div
                class="search-info"
                @click="change(item)">
                <div
                  class="menu-title"
                  v-html="highlightText(item.title.join(' / '))"></div>
                <div
                  class="menu-path"
                  v-html="highlightText(item.path)"></div>
              </div>
              <svg-icon
                icon-class="enter"
                v-show="index === activeIndex"/>
            </div>
          </template>

          <div
            class="empty-state"
            v-else-if="search && options.length === 0">
            <i class="el-icon-search empty-icon"></i>
            <p class="empty-text">
              未找到 "<strong>{{ search }}</strong
              >" 相关菜单
            </p>
            <p class="empty-tip">试试其他关键词或路径</p>
          </div>
        </div>
      </el-scrollbar>

      <div class="search-footer">
        <span class="shortcut-item"> <kbd>↑</kbd><kbd>↓</kbd> 切换 </span>
        <span class="shortcut-item"> <kbd>↵</kbd> 选择 </span>
        <span class="shortcut-item"> <kbd>Esc</kbd> 关闭 </span>
      </div>
    </el-dialog>
  </div>
</template>

<script>
/**
 * 组件名称：HeaderSearch
 * 功能说明：顶部菜单搜索组件，支持标题与路径模糊查询及键盘导航
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

import Fuse from 'fuse.js/dist/fuse.min.js'
import path from 'path'
import { isHttp } from '@/utils/validate'

export default
{
  name: 'HeaderSearch',

  data()
  {
    return {
      search: '',
      options: [],
      searchPool: [],
      activeIndex: -1,
      show: false,
      fuse: undefined,
    }
  },

  computed:
  {
    /**
     * 读取当前主题色
     * @returns {String} 主题色
     */
    theme()
    {
      return this.$store.state.settings.theme
    },

    /**
     * 读取默认路由列表
     * @returns {Array} 路由列表
     */
    routes()
    {
      return this.$store.getters.defaultRoutes
    }
  },

  watch:
  {
    /**
     * 路由变化时重新生成搜索池
     */
    routes()
    {
      this.searchPool = this.generateRoutes(this.routes)
    },

    /**
     * 搜索池变化时初始化模糊搜索
     * @param {Array} list - 搜索池列表
     */
    searchPool(list)
    {
      this.initFuse(list)
    }
  },

  mounted()
  {
    this.searchPool = this.generateRoutes(this.routes)
  },

  methods:
  {
    /**
     * 切换搜索弹窗显示状态
     */
    click()
    {
      this.show = !this.show
      if (this.show)
      {
        this.options = this.searchPool
      }
    },

    /**
     * 弹窗打开后聚焦搜索输入框
     */
    onDialogOpened()
    {
      this.$nextTick(() =>
      {
        this.$refs.headerSearchSelectRef && this.$refs.headerSearchSelectRef.focus()
      })
    },

    /**
     * 关闭弹窗并重置搜索状态
     */
    close()
    {
      this.$refs.headerSearchSelectRef && this.$refs.headerSearchSelectRef.blur()
      this.search = ''
      this.options = this.searchPool
      this.show = false
      this.activeIndex = -1
    },

    /**
     * 选中结果并跳转路由
     * @param {Object} val - 选中项
     */
    change(val)
    {
      const p = val.path
      const query = val.query
      if (isHttp(val.path))
      {
        // http(s):// 路径新窗口打开
        const pindex = p.indexOf('http')
        window.open(p.substr(pindex, p.length), '_blank')
      }
      else
      {
        if (query)
        {
          this.$router.push({ path: p, query: JSON.parse(query) })
        }
        else
        {
          this.$router.push(p)
        }
      }
      this.search = ''
      this.options = this.searchPool
      this.$nextTick(() =>
      {
        this.show = false
      })
    },

    /**
     * 初始化 Fuse 模糊搜索实例
     * @param {Array} list - 搜索池列表
     */
    initFuse(list)
    {
      this.fuse = new Fuse(list, {
        shouldSort: true,
        threshold: 0.2,
        minMatchCharLength: 1,
        keys: [
          {
            name: 'title',
            weight: 0.7,
          },
          {
            name: 'path',
            weight: 0.3,
          },
        ],
      })
    },

    /**
     * 递归生成可搜索的路由列表
     * @param {Array}  routes      - 路由列表
     * @param {String} basePath    - 基础路径
     * @param {Array}  prefixTitle - 前缀标题
     * @returns {Array} 可搜索的路由列表
     */
    generateRoutes(routes, basePath = '/', prefixTitle = [])
    {
      let res = []
      for (const router of routes)
      {
        if (router.hidden)
        {
          continue
        }
        const data = {
          path: !isHttp(router.path) ? path.resolve(basePath, router.path) : router.path,
          title: [...prefixTitle],
          icon: '',
        }
        if (router.meta && router.meta.title)
        {
          data.title = [...data.title, router.meta.title]
          data.icon = router.meta.icon
          if (router.redirect !== 'noRedirect')
          {
            res.push(data)
          }
        }
        if (router.query)
        {
          data.query = router.query
        }
        if (router.children)
        {
          const tempRoutes = this.generateRoutes(router.children, data.path, data.title)
          if (tempRoutes.length >= 1)
          {
            res = [...res, ...tempRoutes]
          }
        }
      }
      return res
    },

    /**
     * 搜索查询，合并路径匹配与模糊匹配结果
     * @param {String} query - 搜索关键词
     */
    querySearch(query)
    {
      this.activeIndex = -1
      if (query !== '')
      {
        const q = query.toLowerCase()
        const pathMatches = this.searchPool.filter((item) => item.path.toLowerCase().includes(q))
        const fuseMatches = this.fuse.search(query).map((item) => item.item)
        const merged = [...pathMatches]
        fuseMatches.forEach((item) =>
        {
          if (!merged.find((m) => m.path === item.path))
          {
            merged.push(item)
          }
        })
        this.options = merged
      }
      else
      {
        this.options = this.searchPool
      }
    },

    /**
     * 计算激活项的样式
     * @param {Number} index - 结果索引
     * @returns {Object} 样式对象
     */
    activeStyle(index)
    {
      if (index !== this.activeIndex) return {}
      return {
        'background-color': this.theme,
        color: '#fff',
      }
    },

    /**
     * 键盘上下键切换选中项
     * @param {String} direction - 方向（up/down）
     */
    navigateResult(direction)
    {
      if (direction === 'up')
      {
        this.activeIndex = this.activeIndex <= 0 ? this.options.length - 1 : this.activeIndex - 1
      }
      else if (direction === 'down')
      {
        this.activeIndex = this.activeIndex >= this.options.length - 1 ? 0 : this.activeIndex + 1
      }
    },

    /**
     * 回车选中当前激活项
     */
    selectActiveResult()
    {
      if (this.options.length > 0 && this.activeIndex >= 0)
      {
        this.change(this.options[this.activeIndex])
      }
    },

    /**
     * 高亮匹配关键词
     * @param {String} text - 原始文本
     * @returns {String} 高亮后的 HTML
     */
    highlightText(text)
    {
      if (!text) return ''
      if (!this.search) return text
      const keyword = this.escapeRegExp(this.search)
      const reg = new RegExp(`(${keyword})`, 'gi')
      return text.replace(reg, '<span class="highlight">$1</span>')
    },

    /**
     * 转义正则特殊字符
     * @param {String} str - 原始字符串
     * @returns {String} 转义后的字符串
     */
    escapeRegExp(str)
    {
      return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    }
  }
}
</script>