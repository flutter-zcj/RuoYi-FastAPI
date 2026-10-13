<style lang="scss" scoped>
.tree-sidebar { display: flex; overflow: hidden; position: relative; flex-shrink: 0; flex-direction: column; width: 220px; background: #fff; border-right: 1px solid #e8eaed; transition: width 0.25s ease;
  &.collapsed { width: 42px; }
  &.resizing { transition: none; will-change: width;
    * { pointer-events: none !important; }
  }
  &.no-initial-transition { transition: none; }
}
.resize-handle { position: absolute; top: 0; right: 0; z-index: 20; width: 6px; height: 100%; background: transparent; cursor: col-resize; transition: background 0.2s;
  &:hover { background: rgba(64, 158, 255, 0.3); }
  &.active { background: rgba(64, 158, 255, 0.5); }
}
.collapse-button-container { position: absolute; top: 50%; right: 0; z-index: 100; display: flex; align-items: center; justify-content: center; width: 15px; height: 20px; background: #fff; transform: translateY(-50%); border-radius: 0 4px 4px 0; box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1); transition: all 0.2s ease;
  .tree-sidebar.collapsed & { right: 0; background: #f7f8fa; border-radius: 0 4px 4px 0; }
  .tree-sidebar.resizing & { pointer-events: none; }
}
.collapse-button { padding: 4px; font-size: 14px; color: #909399; border-radius: 4px; cursor: pointer; transition: all 0.2s;
  &:hover { background: #ecf5ff; color: #409eff; }
}
.tree-header { display: flex; align-items: center; justify-content: space-between; flex-shrink: 0; padding: 0 10px; height: 40px; border-bottom: 1px solid #e8eaed; background: #f7f8fa;
  .tree-title { overflow: hidden; display: flex; align-items: center; gap: 5px; font-size: 13px; font-weight: 600; color: #303133; white-space: nowrap;
    i { color: #409eff; font-size: 14px; }
  }
  .tree-actions { display: flex; align-items: center; gap: 4px; flex-shrink: 0; }
}
.tree-action-icon { padding: 4px; font-size: 14px; color: #909399; border-radius: 4px; cursor: pointer; transition: all 0.2s;
  &:hover { background: #ecf5ff; color: #409eff; }
}
.tree-search { flex-shrink: 0; padding: 10px 10px 4px; }
.tree-wrap { overflow-y: auto; flex: 1; padding: 6px 6px 12px;
  .tree-sidebar.resizing & { overflow: hidden; }
  &::-webkit-scrollbar { width: 4px; }
  &::-webkit-scrollbar-thumb { background: #dcdfe6; border-radius: 4px;
    &:hover { background: #c0c4cc; }
  }
  ::v-deep .el-tree-node__content { height: 32px; margin-bottom: 1px; border-radius: 4px;
    &:hover { background: #f0f7ff; }
  }
  ::v-deep .el-tree-node.is-current > .el-tree-node__content { background: #e6f0fd; color: #409eff; font-weight: 600;
    .node-icon { color: #409eff !important; }
  }
}
.tree-node { display: flex; overflow: hidden; align-items: center; gap: 5px; font-size: 13px;
  .node-icon { flex-shrink: 0; font-size: 14px; color: #f5a623; }
  .node-label { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
}
::v-deep .el-icon-document.node-icon { color: #909399 !important; }
</style>

<template>
  <div
    class="tree-sidebar"
    :class="{
      collapsed: collapsed,
      resizing: isResizing,
      'no-initial-transition': isLoadingFromStorage,
    }"
    :style="{ width: sidebarWidth + 'px' }">
    <!-- 右侧拖动条 -->
    <div
      class="resize-handle"
      :class="{ active: isResizing }"
      v-if="!collapsed"
      @mousedown="startResize"
      @touchstart="startResize"/>
    <div class="tree-header">
      <span class="tree-title" v-show="!collapsed">
        <i :class="titleIconClass"></i> {{ title }}
      </span>
      <div class="tree-actions" v-show="!collapsed">
        <el-tooltip :content="isExpandedAll ? '收起全部' : '展开全部'" placement="right">
          <i
            class="tree-action-icon"
            :class="isExpandedAll ? 'el-icon-arrow-down' : 'el-icon-arrow-up'"
            @click="toggleExpandAll"/>
        </el-tooltip>
        <el-tooltip content="刷新" placement="right">
          <i class="tree-action-icon el-icon-refresh" @click="handleRefresh"/>
        </el-tooltip>
        <slot name="actions"></slot>
      </div>
    </div>

    <!-- 侧边栏展开/收起按钮 -->
    <div class="collapse-button-container">
      <el-tooltip :content="collapsed ? '展开' : '收起'" placement="right">
        <i
          class="collapse-button"
          :class="collapsed ? 'el-icon-d-arrow-right' : 'el-icon-d-arrow-left'"
          @click="toggleCollapsed"/>
      </el-tooltip>
    </div>

    <div
      class="tree-search"
      v-show="!collapsed"
      v-if="showSearch">
      <el-input
        v-model="searchKeyword"
        :placeholder="searchPlaceholder"
        clearable
        size="small"
        prefix-icon="el-icon-search"
        @input="onSearch"/>
    </div>

    <div class="tree-wrap" v-show="!collapsed">
      <el-tree
        ref="treeRef"
        :data="treeData"
        :props="treeProps"
        :expand-on-click-node="expandOnClickNode"
        :filter-node-method="filterNodeMethod"
        :default-expand-all="defaultExpandAll"
        :default-expanded-keys="defaultExpandedKeys"
        :node-key="nodeKey"
        :check-strictly="checkStrictly"
        :show-checkbox="showCheckbox"
        @node-click="onNodeClick"
        @check="onCheck"
        @node-expand="onNodeExpand"
        @node-collapse="onNodeCollapse">
        <span class="tree-node" slot-scope="{ node, data }">
          <slot
            name="node"
            :node="node"
            :data="data">
            <i class="node-icon" :class="data.children && data.children.length ? 'el-icon-folder' : 'el-icon-document'"/>
            <span class="node-label" :title="node.label">{{ node.label }}</span>
          </slot>
        </span>
      </el-tree>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：TreePanel
 * 功能说明：树形侧边栏组件，支持搜索、展开收起、复选、宽度拖拽与本地存储
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Array}    treeData            - 树形数据
 * @prop {String}   title              - 标题
 * @prop {String}   titleIconClass     - 标题图标类名
 * @prop {Boolean}  showSearch         - 是否显示搜索框
 * @prop {String}   searchPlaceholder  - 搜索框占位符
 * @prop {Boolean}  defaultCollapsed   - 是否默认收起侧边栏
 * @prop {Object}   treeProps          - 树配置项
 * @prop {String}   nodeKey            - 节点唯一标识字段
 * @prop {Boolean}  expandOnClickNode  - 是否在点击节点时展开或收起
 * @prop {Boolean}  showCheckbox       - 是否显示复选框
 * @prop {Boolean}  checkStrictly      - 是否严格的遵循父子不互相关联
 * @prop {Boolean}  defaultExpandAll   - 是否默认展开所有节点
 * @prop {Array}    defaultExpandedKeys - 默认展开的节点的key数组
 * @prop {Number}   defaultWidth       - 默认宽度
 * @prop {Number}   collapsedWidth     - 收起时的宽度
 * @prop {Number}   minWidth           - 最小宽度
 * @prop {Number}   maxWidth           - 最大宽度
 * @prop {String}   storageKey         - 本地存储的宽度key
 * @prop {Boolean}  enableStorage      - 是否启用本地存储宽度
 * @prop {Function} filterMethod       - 自定义过滤方法
 *
 * 变量说明：
 *
 * 事件说明：
 * @event collapsed-change    - 收起状态变化
 * @event expanded-all-change - 全部展开状态变化
 * @event search              - 搜索关键词变化
 * @event refresh             - 刷新
 * @event node-click          - 节点点击
 * @event check               - 复选框选中
 * @event node-expand         - 节点展开
 * @event node-collapse       - 节点折叠
 *
 * 插槽说明：
 * actions - 头部操作区插槽
 * node    - 节点内容插槽
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'TreeSidebar',

  props:
  {
    // 树形数据
    treeData: { type: Array, default: () => [] },
    // 标题
    title: { type: String, default: '树形结构' },
    // 标题图标类名
    titleIconClass: { type: String, default: 'el-icon-office-building' },
    // 是否显示搜索框
    showSearch: { type: Boolean, default: true },
    // 搜索框占位符
    searchPlaceholder: { type: String, default: '请输入名称' },
    // 是否默认收起侧边栏
    defaultCollapsed: { type: Boolean, default: false },
    // 树配置项
    treeProps: { type: Object, default: () => ({ children: 'children', label: 'label' }) },
    // 节点唯一标识字段
    nodeKey: { type: String, default: 'id' },
    // 是否在点击节点时展开或收起
    expandOnClickNode: { type: Boolean, default: false },
    // 是否显示复选框
    showCheckbox: { type: Boolean, default: false },
    // 是否严格的遵循父子不互相关联
    checkStrictly: { type: Boolean, default: false },
    // 是否默认展开所有节点
    defaultExpandAll: { type: Boolean, default: false },
    // 默认展开的节点的key数组
    defaultExpandedKeys: { type: Array, default: () => [] },
    // 默认宽度
    defaultWidth: { type: Number, default: 220 },
    // 收起时的宽度
    collapsedWidth: { type: Number, default: 20 },
    // 最小宽度
    minWidth: { type: Number, default: 180 },
    // 最大宽度
    maxWidth: { type: Number, default: 400 },
    // 本地存储的宽度key
    storageKey: { type: String, default: 'tree-sidebar-width' },
    // 是否启用本地存储宽度
    enableStorage: { type: Boolean, default: true },
    // 自定义过滤方法
    filterMethod: { type: Function, default: null }
  },

  data()
  {
    return {
      searchKeyword: '',
      collapsed: this.defaultCollapsed,
      sidebarWidth: this.defaultCollapsed ? this.collapsedWidth : this.defaultWidth,
      isResizing: false,
      startX: 0,
      startWidth: 0,
      saveWidthTimer: null,
      rafId: null,
      isLoadingFromStorage: false,
      expandedAll: this.defaultExpandAll
    }
  },

  computed:
  {
    // 计算当前是否全部展开
    isExpandedAll:
    {
      /**
       * 读取当前展开状态
       * @returns {Boolean} 是否全部展开
       */
      get()
      {
        return this.expandedAll
      },

      /**
       * 设置当前展开状态
       * @param {Boolean} val - 是否全部展开
       */
      set(val)
      {
        this.expandedAll = val
      }
    }
  },

  watch:
  {
    /**
     * 监听收起状态变化并抛出事件
     * @param {Boolean} newVal - 新的收起状态
     * @param {Boolean} oldVal - 旧的收起状态
     */
    collapsed(newVal, oldVal)
    {
      if (newVal !== oldVal)
      {
        this.handleCollapseChange(newVal)
        this.$emit('collapsed-change', newVal)
      }
    },

    // 监听内部展开状态变化，触发实际树的展开/收起
    /**
     * 监听展开状态变化，同步树的展开/收起
     * @param {Boolean} newVal - 新的展开状态
     */
    expandedAll(newVal)
    {
      this.$nextTick(() =>
      {
        if (newVal)
        {
          this.expandAllNodes()
        } else
        {
          this.collapseAllNodes()
        }
      })
      this.$emit('expanded-all-change', newVal)
    },

    // 监听搜索关键词
    /**
     * 监听搜索关键词并过滤树
     * @param {String} val - 搜索关键词
     */
    searchKeyword(val)
    {
      if (this.$refs.treeRef)
      {
        this.$refs.treeRef.filter(val)
        this.$emit('search', val)
      }
    }
  },

  mounted()
  {
    this.isLoadingFromStorage = true
    if (!this.collapsed && this.enableStorage)
    {
      const savedWidth = this.getSavedWidth()
      if (savedWidth !== null)
      {
        this.sidebarWidth = savedWidth
      }
    }
    this.$nextTick(() =>
    {
      this.isLoadingFromStorage = false
    })
    if (this.expandedAll)
    {
      this.$nextTick(() =>
      {
        this.expandAllNodes()
      })
    }
  },

  beforeDestroy()
  {
    this.cleanup()
  },

  methods:
  {
    // 节点过滤方法
    /**
     * 过滤树节点
     * @param {String} value - 过滤关键字
     * @param {Object} data - 节点数据
     * @returns {Boolean} 是否匹配
     */
    filterNodeMethod(value, data)
    {
      if (this.filterMethod)
      {
        return this.filterMethod(value, data)
      }
      if (!value) return true
      return data.label && data.label.indexOf(value) !== -1
    },

    // 清理定时器和动画帧
    /**
     * 清理定时器和动画帧
     */
    cleanup()
    {
      if (this.rafId)
      {
        cancelAnimationFrame(this.rafId)
        this.rafId = null
      }
      if (this.saveWidthTimer)
      {
        clearTimeout(this.saveWidthTimer)
        this.saveWidthTimer = null
      }
    },

    // 处理收起/展开状态变化
    /**
     * 处理收起/展开状态变化
     * @param {Boolean} isCollapsed - 是否收起
     */
    handleCollapseChange(isCollapsed)
    {
      if (isCollapsed)
      {
        this.saveWidthToStorage()
        this.sidebarWidth = this.collapsedWidth
      } else
      {
        const savedWidth = this.getSavedWidth()
        this.sidebarWidth = savedWidth !== null ? savedWidth : this.defaultWidth
      }
    },

    // 获取保存的宽度
    /**
     * 获取保存的宽度
     * @returns {Number|null} 保存的宽度
     */
    getSavedWidth()
    {
      if (!this.enableStorage)
      {
        return null
      }
      try
      {
        const savedWidth = localStorage.getItem(this.storageKey)
        if (savedWidth)
        {
          const width = parseInt(savedWidth, 10)
          if (!isNaN(width) && width >= this.minWidth && width <= this.maxWidth)
          {
            return width
          }
        }
      } catch (error)
      {
        console.warn(
          `Failed to load sidebar width from storage with key ${this.storageKey}:`,
          error
        )
      }
      return null
    },

    // 保存宽度到本地存储
    /**
     * 保存宽度到本地存储
     */
    saveWidthToStorage()
    {
      if (this.collapsed || !this.enableStorage) return
      try
      {
        localStorage.setItem(this.storageKey, this.sidebarWidth.toString())
      } catch (error)
      {
        console.warn(`Failed to save sidebar width to storage with key ${this.storageKey}:`, error)
      }
    },

    // 切换侧边栏收起/展开状态
    /**
     * 切换侧边栏收起/展开状态
     */
    toggleCollapsed()
    {
      this.collapsed = !this.collapsed
    },

    // 切换展开/折叠所有节点
    /**
     * 切换展开/折叠所有节点
     */
    toggleExpandAll()
    {
      this.isExpandedAll = !this.isExpandedAll
    },

    // 展开所有节点
    /**
     * 展开所有节点
     */
    expandAllNodes()
    {
      if (!this.$refs.treeRef) return
      const allNodes = this.getAllNodes(this.$refs.treeRef.root)
      allNodes.forEach((node) =>
      {
        if (node.expanded !== undefined && !node.expanded)
        {
          node.expanded = true
        }
      })
    },

    // 获取所有节点
    /**
     * 获取所有节点
     * @param {Object} rootNode - 根节点
     * @returns {Array} 所有节点
     */
    getAllNodes(rootNode)
    {
      const nodes = []
      const traverse = (node) =>
      {
        if (!node) return
        nodes.push(node)
        if (node.childNodes && node.childNodes.length)
        {
          node.childNodes.forEach((child) => traverse(child))
        }
      }
      traverse(rootNode)
      return nodes
    },

    // 收起所有节点
    /**
     * 收起所有节点
     */
    collapseAllNodes()
    {
      if (!this.$refs.treeRef) return
      const allNodes = this.getAllNodes(this.$refs.treeRef.root)
      allNodes.forEach((node) =>
      {
        if (node.expanded !== undefined && node.expanded)
        {
          node.expanded = false
        }
      })
    },

    // 处理刷新操作
    /**
     * 处理刷新操作
     */
    handleRefresh()
    {
      this.$emit('refresh')
    },

    // 节点点击事件
    /**
     * 节点点击事件
     * @param {Object} data - 节点数据
     * @param {Object} node - 节点对象
     * @param {Event} e - 事件对象
     */
    onNodeClick(data, node, e)
    {
      this.$emit('node-click', data, node, e)
    },

    // 复选框选中事件
    /**
     * 复选框选中事件
     * @param {Object} data - 节点数据
     * @param {Object} checkedInfo - 选中信息
     */
    onCheck(data, checkedInfo)
    {
      this.$emit('check', data, checkedInfo)
    },

    // 节点展开事件
    /**
     * 节点展开事件
     * @param {Object} data - 节点数据
     * @param {Object} node - 节点对象
     * @param {Event} e - 事件对象
     */
    onNodeExpand(data, node, e)
    {
      this.$emit('node-expand', data, node, e)
    },

    // 节点折叠事件
    /**
     * 节点折叠事件
     * @param {Object} data - 节点数据
     * @param {Object} node - 节点对象
     * @param {Event} e - 事件对象
     */
    onNodeCollapse(data, node, e)
    {
      this.$emit('node-collapse', data, node, e)
    },

    // 搜索处理
    /**
     * 搜索处理
     */
    onSearch()
    {
      // 搜索逻辑已在 watch 中处理
    },

    // 设置当前选中的节点
    /**
     * 设置当前选中的节点
     * @param {String|Number} key - 节点key
     */
    setCurrentKey(key)
    {
      if (this.$refs.treeRef)
      {
        this.$refs.treeRef.setCurrentKey(key)
      }
    },

    // 获取当前选中的节点
    /**
     * 获取当前选中的节点
     * @returns {Object|null} 当前选中的节点
     */
    getCurrentNode()
    {
      if (this.$refs.treeRef)
      {
        return this.$refs.treeRef.getCurrentNode()
      }
      return null
    },

    // 获取当前选中的节点的key
    /**
     * 获取当前选中的节点的key
     * @returns {String|Number|null} 当前选中的节点key
     */
    getCurrentKey()
    {
      if (this.$refs.treeRef)
      {
        return this.$refs.treeRef.getCurrentKey()
      }
      return null
    },

    // 设置选中的节点keys（复选框）
    /**
     * 设置选中的节点keys（复选框）
     * @param {Array} keys - 节点key数组
     */
    setCheckedKeys(keys)
    {
      if (this.$refs.treeRef && this.showCheckbox)
      {
        this.$refs.treeRef.setCheckedKeys(keys)
      }
    },

    // 获取选中的节点keys（复选框）
    /**
     * 获取选中的节点keys（复选框）
     * @returns {Array} 选中的节点key数组
     */
    getCheckedKeys()
    {
      if (this.$refs.treeRef && this.showCheckbox)
      {
        return this.$refs.treeRef.getCheckedKeys()
      }
      return []
    },

    // 获取选中的节点（复选框）
    /**
     * 获取选中的节点（复选框）
     * @returns {Array} 选中的节点数组
     */
    getCheckedNodes()
    {
      if (this.$refs.treeRef && this.showCheckbox)
      {
        return this.$refs.treeRef.getCheckedNodes()
      }
      return []
    },

    // 清空搜索
    /**
     * 清空搜索
     */
    clearSearch()
    {
      this.searchKeyword = ''
      if (this.$refs.treeRef)
      {
        this.$refs.treeRef.filter('')
      }
    },

    // 过滤树
    /**
     * 过滤树
     * @param {String} value - 过滤关键字
     */
    filter(value)
    {
      this.searchKeyword = value
    },

    // 开始调整大小
    /**
     * 开始调整大小
     * @param {Event} e - 事件对象
     */
    startResize(e)
    {
      e.preventDefault()
      e.stopPropagation()
      this.isResizing = true
      this.startX = e.type === 'mousedown' ? e.clientX : e.touches[0].clientX
      this.startWidth = this.sidebarWidth

      if (e.type === 'mousedown')
      {
        document.addEventListener('mousemove', this.handleResizeMove)
        document.addEventListener('mouseup', this.stopResize)
      } else
      {
        document.addEventListener('touchmove', this.handleResizeMove, { passive: false })
        document.addEventListener('touchend', this.stopResize)
      }
      this.disableUserSelect()
    },

    // 处理调整大小移动
    /**
     * 处理调整大小移动
     * @param {Event} e - 事件对象
     */
    handleResizeMove(e)
    {
      if (!this.isResizing) return
      if (this.rafId)
      {
        cancelAnimationFrame(this.rafId)
      }
      this.rafId = requestAnimationFrame(() =>
      {
        e.preventDefault()
        e.stopPropagation()
        const clientX = e.type === 'mousemove' ? e.clientX : e.touches[0].clientX
        const deltaX = clientX - this.startX
        const newWidth = this.startWidth + deltaX
        const clampedWidth = Math.max(this.minWidth, Math.min(this.maxWidth, newWidth))
        if (Math.abs(clampedWidth - this.sidebarWidth) >= 1)
        {
          this.sidebarWidth = clampedWidth
        }
      })
    },

    // 停止调整大小
    /**
     * 停止调整大小
     */
    stopResize()
    {
      if (!this.isResizing) return
      this.isResizing = false
      if (this.rafId)
      {
        cancelAnimationFrame(this.rafId)
        this.rafId = null
      }
      this.startX = 0
      this.startWidth = 0
      document.removeEventListener('mousemove', this.handleResizeMove)
      document.removeEventListener('mouseup', this.stopResize)
      document.removeEventListener('touchmove', this.handleResizeMove)
      document.removeEventListener('touchend', this.stopResize)
      this.enableUserSelect()
      this.saveWidthToStorage()
    },

    // 禁用用户选择
    /**
     * 禁用用户选择
     */
    disableUserSelect()
    {
      document.body.style.userSelect = 'none'
      document.body.style.webkitUserSelect = 'none'
      document.body.style.mozUserSelect = 'none'
      document.body.style.msUserSelect = 'none'
    },

    // 启用用户选择
    /**
     * 启用用户选择
     */
    enableUserSelect()
    {
      document.body.style.userSelect = ''
      document.body.style.webkitUserSelect = ''
      document.body.style.mozUserSelect = ''
      document.body.style.msUserSelect = ''
    },

    // 重置宽度到默认值
    /**
     * 重置宽度到默认值
     */
    resetWidth()
    {
      this.sidebarWidth = this.defaultWidth
      this.saveWidthToStorage()
    },

    // 获取当前宽度
    /**
     * 获取当前宽度
     * @returns {Number} 当前宽度
     */
    getCurrentWidth()
    {
      return this.sidebarWidth
    },

    // 设置宽度
    /**
     * 设置宽度
     * @param {Number} width - 宽度
     */
    setWidth(width)
    {
      if (typeof width === 'number' && width >= this.minWidth && width <= this.maxWidth)
      {
        this.sidebarWidth = width
        if (!this.collapsed)
        {
          this.saveWidthToStorage()
        }
      }
    }
  }
}
</script>
