<style lang="scss" scoped>
::v-deep .el-transfer__button { display: block; padding: 12px; margin-left: 0px; border-radius: 50%; }
::v-deep .el-transfer__button:first-child { margin-bottom: 10px; }
.check-line { width: 90%; height: 1px; margin: 3px auto; background-color: #ccc; }
</style>

<template>
  <div class="top-right-btn" :style="style">
    <el-row>
      <el-tooltip
        class="item"
        effect="dark"
        :content="showSearch ? '隐藏搜索' : '显示搜索'"
        placement="top"
        v-if="search">
        <el-button
          size="mini"
          circle
          icon="el-icon-search"
          @click="toggleSearch()"/>
      </el-tooltip>
      <el-tooltip
        class="item"
        effect="dark"
        content="刷新"
        placement="top">
        <el-button
          size="mini"
          circle
          icon="el-icon-refresh"
          @click="refresh()"/>
      </el-tooltip>
      <el-tooltip
        class="item"
        effect="dark"
        content="显隐列"
        placement="top"
        v-if="Object.keys(columns).length > 0">
        <el-button
          size="mini"
          circle
          icon="el-icon-menu"
          v-if="showColumnsType == 'transfer'"
          @click="showColumn()"/>
        <el-dropdown
          trigger="click"
          :hide-on-click="false"
          style="padding-left: 12px"
          v-if="showColumnsType == 'checkbox'">
          <el-button
            size="mini"
            circle
            icon="el-icon-menu"/>
          <el-dropdown-menu slot="dropdown">
            <!-- 全选/反选 按钮 -->
            <el-dropdown-item>
              <el-checkbox
                :indeterminate="isIndeterminate"
                v-model="isChecked"
                @change="toggleCheckAll">
                列展示
              </el-checkbox>
            </el-dropdown-item>
            <div class="check-line"></div>
            <template v-for="(item, key) in columns">
              <el-dropdown-item :key="key">
                <el-checkbox
                  :label="item.label"
                  v-model="item.visible"
                  @change="checkboxChange($event, key)"/>
              </el-dropdown-item>
            </template>
          </el-dropdown-menu>
        </el-dropdown>
      </el-tooltip>
    </el-row>
    <el-dialog
      :title="title"
      :visible.sync="open"
      append-to-body>
      <el-transfer
        :titles="['显示', '隐藏']"
        v-model="value"
        :data="transferData"
        @change="dataChange"></el-transfer>
    </el-dialog>
  </div>
</template>

<script>
/**
 * 组件名称：RightToolbar
 * 功能说明：表格右侧工具栏组件，支持搜索栏显隐、刷新与显隐列配置
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}      showSearch      - 是否显示检索条件
 * @prop {Array|Object} columns         - 显隐列信息（数组格式、对象格式）
 * @prop {Boolean}      search          - 是否显示检索图标
 * @prop {String}       showColumnsType - 显隐列类型（transfer穿梭框、checkbox复选框）
 * @prop {Number}       gutter          - 右外边距
 *
 * 变量说明：
 *
 * 事件说明：
 * @event update:showSearch - 更新检索条件显示状态
 * @event queryTable        - 触发列表查询
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'RightToolbar',

  props:
  {
    /* 是否显示检索条件 */
    showSearch: { type: Boolean, default: true },
    /* 显隐列信息（数组格式、对象格式） */
    columns: { type: [Array, Object], default: () => ({}) },
    /* 是否显示检索图标 */
    search: { type: Boolean, default: true },
    /* 显隐列类型（transfer穿梭框、checkbox复选框） */
    showColumnsType: { type: String, default: 'checkbox' },
    /* 右外边距 */
    gutter: { type: Number, default: 10 }
  },

  data()
  {
    return {
      // 显隐数据
      value: [],
      // 弹出层标题
      title: '显示/隐藏',
      // 是否显示弹出层
      open: false
    }
  },

  computed:
  {
    /**
     * 计算右侧工具栏外边距样式
     * @returns {Object} 样式对象
     */
    style()
    {
      const ret = {}
      if (this.gutter)
      {
        ret.marginRight = `${this.gutter / 2}px`
      }
      return ret
    },

    isChecked:
    {
      /**
       * 读取全选状态
       * @returns {Boolean} 是否全部显示
       */
      get()
      {
        return Array.isArray(this.columns)
          ? this.columns.every((col) => col.visible)
          : Object.values(this.columns).every((col) => col.visible)
      },

      /**
       * 更新全选状态
       */
      set() {}
    },

    /**
     * 计算半选状态
     * @returns {Boolean} 是否处于半选状态
     */
    isIndeterminate()
    {
      return Array.isArray(this.columns)
        ? this.columns.some((col) => col.visible) && !this.isChecked
        : Object.values(this.columns).some((col) => col.visible) && !this.isChecked
    },

    /**
     * 计算穿梭框显隐列数据
     * @returns {Array} 穿梭框数据
     */
    transferData()
    {
      if (Array.isArray(this.columns))
      {
        return this.columns.map((item, index) => ({ key: index, label: item.label }))
      }
      else
      {
        return Object.keys(this.columns).map((key, index) => ({
          key: index,
          label: this.columns[key].label,
        }))
      }
    }
  },

  created()
  {
    if (this.showColumnsType == 'transfer')
    {
      // transfer穿梭显隐列初始默认隐藏列
      if (Array.isArray(this.columns))
      {
        for (let item in this.columns)
        {
          if (this.columns[item].visible === false)
          {
            this.value.push(parseInt(item))
          }
        }
      }
      else
      {
        Object.keys(this.columns).forEach((key, index) => {
          if (this.columns[key].visible === false)
          {
            this.value.push(index)
          }
        })
      }
    }
  },

  methods:
  {
    /**
     * 切换搜索栏显示状态，优先对同级表单执行收起/展开动画
     */
    // 搜索
    toggleSearch()
    {
      let el = this.$el
      let formEl = null
      while ((el = el.parentElement) && el !== document.body)
      {
        if ((formEl = el.querySelector('.el-form'))) break
      }
      if (!formEl) return this.$emit('update:showSearch', !this.showSearch)
      this._animateSearch(formEl, this.showSearch)
    },

    /**
     * 执行搜索栏展开/收起动画
     * @param {HTMLElement} el - 搜索表单元素
     * @param {Boolean} isHide - 是否隐藏
     */
    // 搜索栏动画
    _animateSearch(el, isHide)
    {
      const DURATION = 260
      const TRANSITION = 'max-height 0.25s ease, opacity 0.2s ease'
      const clear = () =>
        Object.assign(el.style, { transition: '', maxHeight: '', opacity: '', overflow: '' })
      Object.assign(el.style, { overflow: 'hidden', transition: '' })
      if (isHide)
      {
        Object.assign(el.style, {
          maxHeight: el.scrollHeight + 'px',
          opacity: '1',
          transition: TRANSITION,
        })
        requestAnimationFrame(() => Object.assign(el.style, { maxHeight: '0', opacity: '0' }))
        setTimeout(() => {
          this.$emit('update:showSearch', false)
          clear()
        }, DURATION)
      }
      else
      {
        this.$emit('update:showSearch', true)
        this.$nextTick(() => {
          Object.assign(el.style, { maxHeight: '0', opacity: '0' })
          requestAnimationFrame(() =>
            requestAnimationFrame(() => {
              Object.assign(el.style, {
                transition: TRANSITION,
                maxHeight: el.scrollHeight + 'px',
                opacity: '1',
              })
            })
          )
          setTimeout(clear, DURATION)
        })
      }
    },

    /**
     * 触发列表刷新查询
     */
    // 刷新
    refresh()
    {
      this.$emit('queryTable')
    },

    /**
     * 穿梭框数据变化时更新显隐列
     * @param {Array} data - 穿梭框右侧数据
     */
    // 右侧列表元素变化
    dataChange(data)
    {
      if (Array.isArray(this.columns))
      {
        for (let item in this.columns)
        {
          const key = this.columns[item].key
          this.columns[item].visible = !data.includes(key)
        }
      }
      else
      {
        Object.keys(this.columns).forEach((key, index) => {
          this.columns[key].visible = !data.includes(index)
        })
      }
    },

    /**
     * 打开显隐列配置弹窗
     */
    // 打开显隐列dialog
    showColumn()
    {
      this.open = true
    },

    /**
     * 单个显隐列勾选变化
     * @param {Boolean} event - 勾选状态
     * @param {Number|String} key - 列标识
     */
    // 单勾选
    checkboxChange(event, key)
    {
      if (Array.isArray(this.columns))
      {
        this.columns.filter((item) => item.key == key)[0].visible = event
      }
      else
      {
        this.columns[key].visible = event
      }
    },

    /**
     * 切换全选/反选
     */
    // 切换全选/反选
    toggleCheckAll()
    {
      const newValue = !this.isChecked
      if (Array.isArray(this.columns))
      {
        this.columns.forEach((col) => (col.visible = newValue))
      }
      else
      {
        Object.values(this.columns).forEach((col) => (col.visible = newValue))
      }
    }
  }
}
</script>