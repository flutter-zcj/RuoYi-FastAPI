<style lang="scss" scoped></style>

<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      v-show="showSearch"
      label-width="68px">
      <el-form-item label="操作地址" prop="operIp">
        <el-input
          v-model="queryParams.operIp"
          placeholder="请输入操作地址"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="系统模块" prop="title">
        <el-input
          v-model="queryParams.title"
          placeholder="请输入系统模块"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="操作人员" prop="operName">
        <el-input
          v-model="queryParams.operName"
          placeholder="请输入操作人员"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="类型" prop="businessType">
        <el-select
          v-model="queryParams.businessType"
          placeholder="操作类型"
          clearable
          style="width: 240px">
          <el-option
            v-for="dict in dict.type.sys_oper_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="操作状态"
          clearable
          style="width: 240px">
          <el-option
            v-for="dict in dict.type.sys_common_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="操作时间">
        <el-date-picker
          v-model="dateRange"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
          :aria-label="`日期范围（${getDisplayTimezone()}）`"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery">搜索</el-button>
        <el-button
          icon="el-icon-refresh"
          size="mini"
          @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row class="mb8" :gutter="10">
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          v-hasPermi="['monitor:operlog:remove']"
          @click="handleDelete">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          v-hasPermi="['monitor:operlog:remove']"
          @click="handleClean">清空</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          v-hasPermi="['monitor:operlog:export']"
          @click="handleExport">导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      ref="tables"
      v-loading="loading"
      :data="list"
      :default-sort="defaultSort"
      @selection-change="handleSelectionChange"
      @sort-change="handleSortChange">
      <el-table-column
        type="selection"
        width="50"
        align="center"/>
      <el-table-column
        label="日志编号"
        align="center"
        prop="operId"/>
      <el-table-column
        label="系统模块"
        align="center"
        prop="title"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="操作类型"
        align="center"
        prop="businessType">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_oper_type" :value="scope.row.businessType"/>
        </template>
      </el-table-column>
      <el-table-column
        label="操作人员"
        align="center"
        prop="operName"
        width="110"
        :show-overflow-tooltip="true"
        sortable="custom"
        :sort-orders="['descending', 'ascending']"/>
      <el-table-column
        label="操作地址"
        align="center"
        prop="operIp"
        width="130"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="操作地点"
        align="center"
        prop="operLocation"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="操作状态"
        align="center"
        prop="status">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_common_status" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column
        label="操作日期"
        align="center"
        prop="operTime"
        width="160"
        sortable="custom"
        :sort-orders="['descending', 'ascending']">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.operTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="消耗时间"
        align="center"
        prop="costTime"
        width="110"
        :show-overflow-tooltip="true"
        sortable="custom"
        :sort-orders="['descending', 'ascending']">
        <template slot-scope="scope">
          <span>{{ scope.row.costTime }}毫秒</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-view"
            v-hasPermi="['monitor:operlog:query']"
            @click="handleDetail(scope.row, scope.index)">详细</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"/>

    <operlog-detail :visible.sync="detailVisible" :row="detailRow"/>
  </div>
</template>

<script>
/**
 * 组件名称：Operlog
 * 功能说明：操作日志页面，支持操作日志查询、删除、清空、导出与详情查看
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} loading       - 遮罩层
 * @data {Array}   ids           - 选中数组
 * @data {Boolean} multiple      - 非多个禁用
 * @data {Boolean} showSearch    - 显示搜索条件
 * @data {Number}  total         - 总条数
 * @data {Array}   list          - 表格数据
 * @data {Boolean} detailVisible - 是否显示弹出层
 * @data {Object}  detailRow     - 详情行数据
 * @data {Array}   dateRange     - 日期范围
 * @data {Object}  defaultSort   - 默认排序
 * @data {Object}  form          - 表单参数
 * @data {Object}  queryParams   - 查询参数
 *
 * 事件说明：
 * @event getList               - 查询登录日志
 * @event handleDetail          - 详细按钮操作
 * @event handleQuery           - 搜索按钮操作
 * @event resetQuery            - 重置按钮操作
 * @event handleSelectionChange - 多选框选中数据
 * @event handleSortChange      - 排序触发事件
 * @event handleDelete          - 删除按钮操作
 * @event handleClean           - 清空按钮操作
 * @event handleExport          - 导出按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getDisplayTimezone } from '@/utils/time'
import OperlogDetail from './detail'
import { list, delOperlog, cleanOperlog } from '@/api/monitor/operlog'

export default
{
  name: 'Operlog',

  components: { OperlogDetail },
  dicts: ['sys_oper_type', 'sys_common_status'],

  data()
  {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 表格数据
      list: [],
      // 是否显示弹出层
      detailVisible: false,
      detailRow: {},
      // 日期范围
      dateRange: [],
      // 默认排序
      defaultSort: { prop: 'operTime', order: 'descending' },
      // 表单参数
      form: {},
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        operIp: undefined,
        title: undefined,
        operName: undefined,
        businessType: undefined,
        status: undefined,
      },
    }
  },

  created()
  {
    // 时区切换后重新解释日期筛选，保留当前表单。
    this.$watch(getDisplayTimezone, () => {
      if (this.dateRange && this.dateRange.length) this.handleQuery()
    })
    this.getList()
  },

  methods:
  {
    /**
     * 获取显示时区
     */
    getDisplayTimezone,

    /**
     * 查询登录日志
     */
    getList()
    {
      this.loading = true
      list(this.addDateRange(this.queryParams, this.dateRange)).then((response) => {
        this.list = response.rows
        this.total = response.total
        this.loading = false
      })
    },

    /**
     * 详细按钮操作
     */
    handleDetail(row)
    {
      this.detailRow = row
      this.detailVisible = true
    },

    /**
     * 搜索按钮操作
     */
    handleQuery()
    {
      this.queryParams.pageNum = 1
      this.getList()
    },

    /**
     * 重置按钮操作
     */
    resetQuery()
    {
      this.dateRange = []
      this.resetForm('queryForm')
      this.queryParams.pageNum = 1
      this.$refs.tables.sort(this.defaultSort.prop, this.defaultSort.order)
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的行数据
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.operId)
      this.multiple = !selection.length
    },

    /**
     * 排序触发事件
     */
    handleSortChange(column, prop, order)
    {
      this.queryParams.orderByColumn = column.prop
      this.queryParams.isAsc = column.order
      this.getList()
    },

    /**
     * 删除按钮操作
     */
    handleDelete(row)
    {
      const operIds = row.operId || this.ids
      this.$modal
        .confirm('是否确认删除日志编号为"' + operIds + '"的数据项？')
        .then(function () {
          return delOperlog(operIds)
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('删除成功')
        })
        .catch(() => {})
    },

    /**
     * 清空按钮操作
     */
    handleClean()
    {
      this.$modal
        .confirm('是否确认清空所有操作日志数据项？')
        .then(function () {
          return cleanOperlog()
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('清空成功')
        })
        .catch(() => {})
    },

    /**
     * 导出按钮操作
     */
    handleExport()
    {
      this.download(
        'monitor/operlog/export',
        {
          ...this.queryParams,
        },
        `operlog_${new Date().getTime()}.xlsx`
      )
    },
  },
}
</script>