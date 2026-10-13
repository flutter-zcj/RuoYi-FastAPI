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
      <el-form-item label="登录地址" prop="ipaddr">
        <el-input
          v-model="queryParams.ipaddr"
          placeholder="请输入登录地址"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="用户名称" prop="userName">
        <el-input
          v-model="queryParams.userName"
          placeholder="请输入用户名称"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="登录状态"
          clearable
          style="width: 240px">
          <el-option
            v-for="dict in dict.type.sys_common_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="登录时间">
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
          v-hasPermi="['monitor:logininfor:remove']"
          @click="handleDelete">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          v-hasPermi="['monitor:logininfor:remove']"
          @click="handleClean">清空</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-unlock"
          size="mini"
          :disabled="single"
          v-hasPermi="['monitor:logininfor:unlock']"
          @click="handleUnlock">解锁</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          v-hasPermi="['monitor:logininfor:export']"
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
        width="55"
        align="center"/>
      <el-table-column
        label="访问编号"
        align="center"
        prop="infoId"/>
      <el-table-column
        label="用户名称"
        align="center"
        prop="userName"
        :show-overflow-tooltip="true"
        sortable="custom"
        :sort-orders="['descending', 'ascending']"/>
      <el-table-column
        label="登录地址"
        align="center"
        prop="ipaddr"
        width="130"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="登录地点"
        align="center"
        prop="loginLocation"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="浏览器"
        align="center"
        prop="browser"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="操作系统"
        align="center"
        prop="os"/>
      <el-table-column
        label="登录状态"
        align="center"
        prop="status">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_common_status" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column
        label="操作信息"
        align="center"
        prop="msg"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="登录日期"
        align="center"
        prop="loginTime"
        sortable="custom"
        :sort-orders="['descending', 'ascending']"
        width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.loginTime) }}</span>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"/>
  </div>
</template>

<script>
/**
 * 组件名称：Logininfor
 * 功能说明：登录日志页面，支持登录日志查询、删除、清空、解锁与导出
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} loading     - 遮罩层
 * @data {Array}   ids         - 选中数组
 * @data {Boolean} single      - 非单个禁用
 * @data {Boolean} multiple    - 非多个禁用
 * @data {String}  selectName  - 选择用户名
 * @data {Boolean} showSearch  - 显示搜索条件
 * @data {Number}  total       - 总条数
 * @data {Array}   list        - 表格数据
 * @data {Array}   dateRange   - 日期范围
 * @data {Object}  defaultSort - 默认排序
 * @data {Object}  queryParams - 查询参数
 *
 * 事件说明：
 * @event getList               - 查询登录日志列表
 * @event handleQuery           - 搜索按钮操作
 * @event resetQuery            - 重置按钮操作
 * @event handleSelectionChange - 多选框选中数据
 * @event handleSortChange      - 排序触发事件
 * @event handleDelete          - 删除按钮操作
 * @event handleClean           - 清空按钮操作
 * @event handleUnlock          - 解锁按钮操作
 * @event handleExport          - 导出按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getDisplayTimezone } from '@/utils/time'
import { list, delLogininfor, cleanLogininfor, unlockLogininfor } from '@/api/monitor/logininfor'

export default
{
  name: 'Logininfor',

  dicts: ['sys_common_status'],

  data()
  {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 选择用户名
      selectName: '',
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 表格数据
      list: [],
      // 日期范围
      dateRange: [],
      // 默认排序
      defaultSort: { prop: 'loginTime', order: 'descending' },
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        ipaddr: undefined,
        userName: undefined,
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
     * 查询登录日志列表
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
      this.ids = selection.map((item) => item.infoId)
      this.single = selection.length != 1
      this.multiple = !selection.length
      this.selectName = selection.map((item) => item.userName)
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
      const infoIds = row.infoId || this.ids
      this.$modal
        .confirm('是否确认删除访问编号为"' + infoIds + '"的数据项？')
        .then(function () {
          return delLogininfor(infoIds)
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
        .confirm('是否确认清空所有登录日志数据项？')
        .then(function () {
          return cleanLogininfor()
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('清空成功')
        })
        .catch(() => {})
    },

    /**
     * 解锁按钮操作
     */
    handleUnlock()
    {
      const username = this.selectName
      this.$modal
        .confirm('是否确认解锁用户"' + username + '"数据项?')
        .then(function () {
          return unlockLogininfor(username)
        })
        .then(() => {
          this.$modal.msgSuccess('用户' + username + '解锁成功')
        })
        .catch(() => {})
    },

    /**
     * 导出按钮操作
     */
    handleExport()
    {
      this.download(
        'monitor/logininfor/export',
        {
          ...this.queryParams,
        },
        `logininfor_${new Date().getTime()}.xlsx`
      )
    },
  },
}
</script>