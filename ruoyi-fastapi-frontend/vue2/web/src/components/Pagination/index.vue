<style lang="scss" scoped>
.pagination-container { background: #fff; }
.pagination-container.hidden { display: none; }
</style>

<template>
  <div class="pagination-container" :class="{ hidden: hidden }">
    <el-pagination
      :background="background"
      :current-page.sync="currentPage"
      :page-size.sync="pageSize"
      :layout="layout"
      :page-sizes="pageSizes"
      :pager-count="pagerCount"
      :total="total"
      v-bind="$attrs"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"/>
  </div>
</template>

<script>
/**
 * 组件名称：Pagination
 * 功能说明：分页组件，支持页码切换、每页条数切换与列表自动滚动
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Number}  total      - 总条数
 * @prop {Number}  page       - 当前页码
 * @prop {Number}  limit      - 每页条数
 * @prop {Array}   pageSizes  - 每页条数选项
 * @prop {Number}  pagerCount - 页码按钮数量
 * @prop {String}  layout     - 分页布局
 * @prop {Boolean} background - 是否显示背景色
 * @prop {Boolean} autoScroll - 是否自动滚动到顶部
 * @prop {Boolean} hidden     - 是否隐藏
 *
 * 变量说明：
 *
 * 事件说明：
 * @event handleSizeChange    - 每页条数变更
 * @event handleCurrentChange - 当前页码变更
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { scrollTo } from '@/utils/scroll-to'

export default
{
  name: 'Pagination',

  props:
  {
    total: { required: true, type: Number },
    page: { type: Number, default: 1 },
    limit: { type: Number, default: 20 },
    pageSizes: { type: Array, default() { return [10, 20, 30, 50] } },
    // 移动端页码按钮的数量端默认值5
    pagerCount: { type: Number, default: document.body.clientWidth < 992 ? 5 : 7 },
    layout: { type: String, default: 'total, sizes, prev, pager, next, jumper' },
    background: { type: Boolean, default: true },
    autoScroll: { type: Boolean, default: true },
    hidden: { type: Boolean, default: false }
  },

  data()
  {
    return {}
  },

  computed:
  {
    currentPage:
    {
      /**
       * 读取当前页码
       * @returns {Number} 当前页码
       */
      get()
      {
        return this.page
      },

      /**
       * 更新当前页码并向上抛出事件
       * @param {Number} val - 新的页码
       */
      set(val)
      {
        this.$emit('update:page', val)
      }
    },

    pageSize:
    {
      /**
       * 读取每页条数
       * @returns {Number} 每页条数
       */
      get()
      {
        return this.limit
      },

      /**
       * 更新每页条数并向上抛出事件
       * @param {Number} val - 新的每页条数
       */
      set(val)
      {
        this.$emit('update:limit', val)
      }
    }
  },

  methods:
  {
    /**
     * 每页条数变更，必要时重置页码，并在开启自动滚动时滚动到顶部
     * @param {Number} val - 新的每页条数
     */
    handleSizeChange(val)
    {
      if (this.currentPage * val > this.total)
      {
        this.currentPage = 1
      }
      this.$emit('pagination', { page: this.currentPage, limit: val })
      if (this.autoScroll)
      {
        scrollTo(0, 800)
      }
    },

    /**
     * 当前页码变更，并向上抛出分页事件
     * @param {Number} val - 新的页码
     */
    handleCurrentChange(val)
    {
      this.$emit('pagination', { page: val, limit: this.pageSize })
      if (this.autoScroll)
      {
        scrollTo(0, 800)
      }
    }
  }
}
</script>