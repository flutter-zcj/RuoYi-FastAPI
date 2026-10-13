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
      <el-form-item label="任务编号" prop="jobId">
        <el-input
          v-model="queryParams.jobId"
          placeholder="请输入任务编号"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="执行编号" prop="executionId">
        <el-input
          v-model="queryParams.executionId"
          placeholder="请输入执行ID"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="任务名称" prop="jobName">
        <el-input
          v-model="queryParams.jobName"
          placeholder="请输入任务名称"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="任务组名" prop="jobGroup">
        <el-input
          v-model="queryParams.jobGroup"
          placeholder="请输入任务组名"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="执行状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="请选择执行状态"
          clearable
          style="width: 240px">
          <el-option
            v-for="dict in dict.type.sys_common_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value" />
        </el-select>
      </el-form-item>
      <el-form-item label="执行时间">
        <el-date-picker
          v-model="dateRange"
          :aria-label="'执行日期（' + displayTimezone + '）'"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
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

    <el-row
      class="mb8"
      :gutter="10">
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['monitor:job:remove']">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          @click="handleClean"
          v-hasPermi="['monitor:job:remove']">清空</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['monitor:job:export']">导出</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-close"
          size="mini"
          @click="handleClose">关闭</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="jobLogList"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55"
        align="center" />
      <el-table-column
        label="日志编号"
        width="80"
        align="center"
        prop="jobLogId" />
      <el-table-column
        label="任务名称"
        align="center"
        prop="jobName"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="任务组名"
        align="center"
        prop="jobGroup"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="调用目标字符串"
        align="center"
        prop="invokeTarget"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="日志信息"
        align="center"
        prop="jobMessage"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="执行状态"
        align="center"
        prop="status">
        <template slot-scope="scope">
          <dict-tag
            :options="dict.type.sys_common_status"
            :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column
        label="执行时间"
        align="center"
        prop="createTime"
        width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
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
            @click="handleView(scope.row)"
            v-hasPermi="['monitor:job:query']">详细</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList" />

    <!-- 调度日志详细 -->
    <job-log-detail
      :visible.sync="open"
      :row="form"
      type="log" />
  </div>
</template>

<script>
/**
 * 组件名称：JobLog
 * 功能说明：调度日志管理页面，支持日志查询、详细查看、删除与清空
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {Boolean} loading     - 遮罩层
 * @data {Array}   ids         - 选中数组
 * @data {Boolean} multiple    - 非多个禁用
 * @data {Boolean} showSearch  - 显示搜索条件
 * @data {Number}  total       - 总条数
 * @data {Array}   jobLogList  - 调度日志表格数据
 * @data {Boolean} open        - 是否显示弹出层
 * @data {Array}   dateRange   - 日期范围
 * @data {Object}  form        - 表单参数
 * @data {Object}  queryParams - 查询参数
 *
 * 事件说明：
 * @event getList               - 查询调度日志列表
 * @event handleClose           - 返回按钮
 * @event handleQuery           - 搜索按钮操作
 * @event resetQuery            - 重置按钮操作
 * @event handleSelectionChange - 多选框选中数据
 * @event handleView            - 详细按钮操作
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
import { listJobLog, delJobLog, cleanJobLog } from '@/api/monitor/jobLog'
import JobLogDetail from './detail'

export default
{
  name: 'JobLog',

  components:
  {
    JobLogDetail
  },

  dicts: ['sys_common_status'],

  /**
   * 组件数据
   * @returns {Object} 组件响应式数据
   */
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
      // 调度日志表格数据
      jobLogList: [],
      // 是否显示弹出层
      open: false,
      // 日期范围
      dateRange: [],
      // 表单参数
      form: {},
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        jobId: undefined,
        executionId: undefined,
        jobName: undefined,
        jobGroup: undefined,
        status: undefined,
      },
    }
  },

  computed:
  {
    /**
     * 当前路由对象
     * @returns {Object} 路由对象
     */
    route()
    {
      return this.$route
    },

    /**
     * 路由查询参数
     * @returns {Array} 路由参数与查询值
     */
    routeQuery()
    {
      return [this.route.params.jobId, this.route.query.executionId]
    },

    /**
     * 展示时区
     * @returns {String} 时区标识
     */
    displayTimezone()
    {
      return getDisplayTimezone()
    },
  },

  watch:
  {
    /**
     * 监听路由参数变化并重新查询
     */
    routeQuery:
    {
      handler([jobId, executionId])
      {
        this.queryParams.jobId = jobId && jobId != 0 ? jobId : undefined
        this.queryParams.executionId = executionId || undefined
        this.queryParams.jobName = undefined
        this.queryParams.jobGroup = undefined
        this.handleQuery()
      },
      immediate: true,
    },

    /**
     * 监听展示时区变化并重新查询
     */
    displayTimezone()
    {
      if (this.dateRange?.length)
      {
        this.handleQuery()
      }
    },
  },

  methods:
  {
    /**
     * 查询调度日志列表
     */
    getList()
    {
      this.loading = true
      const query = {
        ...this.queryParams,
        jobId: this.queryParams.jobId || undefined,
        executionId: this.queryParams.executionId || undefined,
      }
      listJobLog(this.addDateRange(query, this.dateRange))
        .then((response) =>
        {
          this.jobLogList = response.rows
          this.total = response.total
        })
        .finally(() =>
        {
          this.loading = false
        })
    },

    /**
     * 返回按钮
     */
    handleClose()
    {
      const obj = { path: '/monitor/job' }
      this.$tab.closeOpenPage(obj)
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
      this.handleQuery()
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的数据集合
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.jobLogId)
      this.multiple = !selection.length
    },

    /**
     * 详细按钮操作
     * @param {Object} row - 当前行数据
     */
    handleView(row)
    {
      this.open = true
      this.form = row
    },

    /**
     * 删除按钮操作
     * @param {Object} row - 当前行数据
     */
    handleDelete(row)
    {
      const jobLogIds = this.ids
      this.$modal
        .confirm('是否确认删除调度日志编号为"' + jobLogIds + '"的数据项？')
        .then(function ()
        {
          return delJobLog(jobLogIds)
        })
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('删除成功')
        })
        .catch(() =>
        {})
    },

    /**
     * 清空按钮操作
     */
    handleClean()
    {
      this.$modal
        .confirm('是否确认清空所有调度日志数据项？')
        .then(function ()
        {
          return cleanJobLog()
        })
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('清空成功')
        })
        .catch(() =>
        {})
    },

    /**
     * 导出按钮操作
     */
    handleExport()
    {
      this.download(
        'monitor/jobLog/export',
        this.addDateRange(
          {
            ...this.queryParams,
            jobId: this.queryParams.jobId || undefined,
            executionId: this.queryParams.executionId || undefined,
          },
          this.dateRange
        ),
        `job_log_${new Date().getTime()}.xlsx`
      )
    },
  },
}
</script>