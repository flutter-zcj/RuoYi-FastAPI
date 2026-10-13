<style lang="scss" scoped>
.form-help { color: #909399; font-size: 12px; line-height: 1.6; margin-top: 4px; }
.field-unit { margin: 0 16px 0 8px; }
</style>

<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      v-show="showSearch"
      label-width="68px">
      <el-form-item label="任务名称" prop="jobName">
        <el-input
          v-model="queryParams.jobName"
          placeholder="请输入任务名称"
          clearable
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="任务组名" prop="jobGroup">
        <el-input
          v-model="queryParams.jobGroup"
          placeholder="请输入任务组名"
          clearable
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="任务状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="请选择任务状态"
          clearable>
          <el-option
            v-for="dict in dict.type.sys_job_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value" />
        </el-select>
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
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['monitor:job:add']">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['monitor:job:edit']">修改</el-button>
      </el-col>
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
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['monitor:job:export']">导出</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          icon="el-icon-s-operation"
          size="mini"
          @click="handleJobLog"
          v-hasPermi="['monitor:job:query']">日志</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          icon="el-icon-tickets"
          size="mini"
          @click="handleRuntime('executions')"
          v-hasPermi="['monitor:job:query']">运行记录</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="jobList"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55"
        align="center" />
      <el-table-column
        label="任务编号"
        width="100"
        align="center"
        prop="jobId" />
      <el-table-column
        label="任务名称"
        min-width="120"
        align="center"
        :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <a
            class="link-type"
            style="cursor: pointer"
            @click="handleView(scope.row)">{{ scope.row.jobName }}</a>
        </template>
      </el-table-column>
      <el-table-column
        label="任务组名"
        min-width="100"
        align="center"
        prop="jobGroup"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="调用目标字符串"
        min-width="130"
        align="center"
        prop="invokeTarget"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="cron执行表达式"
        min-width="130"
        align="center"
        prop="cronExpression"
        :show-overflow-tooltip="true" />
      <el-table-column
        label="cron时区"
        align="center"
        prop="timeZone"
        width="150" />
      <el-table-column
        label="状态"
        align="center"
        width="80">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-value="0"
            inactive-value="1"
            :aria-label="'任务 ' + scope.row.jobName + ' 的启停状态'"
            :disabled="changingIds.includes(scope.row.jobId)"
            @change="handleStatusChange(scope.row)"></el-switch>
        </template>
      </el-table-column>
      <el-table-column
        label="同步状态"
        align="center"
        width="100">
        <template slot-scope="scope">
          <el-button
            v-if="canQueryRuntime"
            size="mini"
            type="text"
            @click="handleRuntime('sync', scope.row)">
            {{ (syncStates[scope.row.syncStatus] || syncStates.pending).label }}
          </el-button>
          <el-tag
            v-else
            size="small"
            :type="(syncStates[scope.row.syncStatus] || syncStates.pending).type">
            {{ (syncStates[scope.row.syncStatus] || syncStates.pending).label }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        fixed="right"
        width="180"
        class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['monitor:job:edit']">修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['monitor:job:remove']">删除</el-button>
          <el-dropdown
            size="mini"
            @command="(command) => handleCommand(command, scope.row)"
            v-hasPermi="['monitor:job:changeStatus', 'monitor:job:query']">
            <el-button
              size="mini"
              type="text"
              icon="el-icon-d-arrow-right">更多</el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                command="handleRun"
                icon="el-icon-caret-right"
                :disabled="runningIds.includes(scope.row.jobId)"
                v-hasPermi="['monitor:job:changeStatus']">执行一次</el-dropdown-item>
              <el-dropdown-item
                command="handleJobLog"
                icon="el-icon-s-operation"
                v-hasPermi="['monitor:job:query']">调度日志</el-dropdown-item>
              <el-dropdown-item
                command="handleExecutions"
                icon="el-icon-tickets"
                v-hasPermi="['monitor:job:query']">执行记录</el-dropdown-item>
              <el-dropdown-item
                command="handleSync"
                icon="el-icon-refresh"
                v-hasPermi="['monitor:job:query']">同步状态</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList" />

    <!-- 添加或修改定时任务对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="open"
      width="800px"
      append-to-body>
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        label-width="120px">
        <el-row>
          <el-col :span="24">
            <el-form-item label="任务名称" prop="jobName">
              <el-input v-model="form.jobName" placeholder="请输入任务名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="任务分组" prop="jobGroup">
              <el-input v-model="form.jobGroup" placeholder="请输入业务分组" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item prop="jobExecutor">
              <span slot="label">
                任务执行器
                <el-tooltip placement="top">
                  <div slot="content">调用方法为异步函数时此选项无效</div>
                  <i class="el-icon-question"></i>
                </el-tooltip>
              </span>
              <el-select v-model="form.jobExecutor" placeholder="请选择任务执行器">
                <el-option
                  v-for="dict in dict.type.sys_job_executor"
                  :key="dict.value"
                  :label="dict.label"
                  :value="dict.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="调度存储" prop="jobStore">
              <el-select v-model="form.jobStore" placeholder="请选择调度存储">
                <el-option
                  v-for="dict in dict.type.sys_job_store"
                  :key="dict.value"
                  :label="dict.label"
                  :value="dict.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="cron时区" prop="timeZone">
              <el-select
                v-model="form.timeZone"
                filterable
                allow-create
                placeholder="选择IANA时区">
                <el-option
                  v-for="zone in timeZoneOptions"
                  :key="zone"
                  :label="zone"
                  :value="zone" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item prop="invokeTarget">
              <span slot="label">
                调用方法
                <el-tooltip placement="top">
                  <div slot="content">调用示例：module_task.scheduler_test.job</div>
                  <i class="el-icon-question"></i>
                </el-tooltip>
              </span>
              <el-input v-model="form.invokeTarget" placeholder="请输入调用目标字符串" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="位置参数" prop="jobArgs">
              <json-editor
                v-model="form.jobArgs"
                label="位置参数"
                value-type="array"
                :validate="(value) => parseJobParameter(value, 'jobArgs')" />
              <div class="form-help">使用 JSON 数组，无参数时填写 []。</div>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="关键字参数" prop="jobKwargs">
              <json-editor
                v-model="form.jobKwargs"
                label="关键字参数"
                value-type="object"
                :validate="(value) => parseJobParameter(value, 'jobKwargs')" />
              <div class="form-help">使用 JSON 对象，无参数时填写 {}。</div>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="cron表达式" prop="cronExpression">
              <el-input v-model="form.cronExpression" placeholder="请输入cron执行表达式">
                <template slot="append">
                  <el-button type="primary" @click="handleShowCron">
                    生成表达式
                    <i class="el-icon-time el-icon--right"></i>
                  </el-button>
                </template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24" v-if="form.jobId !== undefined">
            <el-form-item label="状态">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in dict.type.sys_job_status"
                  :key="dict.value"
                  :label="dict.value">{{ dict.label }}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="允许延迟" prop="misfireGraceTime">
              <el-input-number
                v-if="!unlimitedDelay"
                v-model="form.misfireGraceTime"
                :min="1"
                :max="2147483647"
                :precision="0"
                controls-position="right"
                aria-label="允许延迟秒数" />
              <span class="field-unit" v-if="!unlimitedDelay">秒</span>
              <el-checkbox v-model="unlimitedDelay">不限延迟</el-checkbox>
              <div class="form-help">超过允许延迟的计划会跳过；不限延迟时允许补跑过期计划。</div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="积压处理" prop="coalesce">
              <el-radio-group v-model="form.coalesce" size="small">
                <el-radio-button :label="false">逐次执行</el-radio-button>
                <el-radio-button :label="true">仅最近一次</el-radio-button>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="最大并发数" prop="maxInstances">
              <el-input-number
                v-model="form.maxInstances"
                :min="1"
                :max="2147483647"
                :precision="0"
                controls-position="right" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item>
              <div class="form-help">
                积压处理仍受允许延迟限制；定时和手动执行共同遵守并发上限，超限时跳过并记录原因。
              </div>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <el-button
          type="primary"
          :loading="submitting"
          @click="submitForm">确 定</el-button>
        <el-button
          :disabled="submitting"
          @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="Cron表达式生成器"
      :visible.sync="openCron"
      append-to-body
      destroy-on-close
      class="scrollbar">
      <crontab
        @hide="openCron = false"
        @fill="crontabFill"
        :expression="expression"
        :time-zone="form.timeZone"></crontab>
    </el-dialog>

    <!-- 任务详细 -->
    <job-detail
      :visible.sync="openView"
      :row="form"
      type="job" />
    <job-runtime
      :visible.sync="runtimeOpen"
      :initial-tab="runtimeContext.tab"
      :job-id="runtimeContext.jobId"
      :execution-id="runtimeContext.executionId"
      @sync-updated="getList" />
  </div>
</template>

<script>
/**
 * 组件名称：Job
 * 功能说明：定时任务管理页面，支持任务查询、新增、修改、删除、启停与运行记录查看
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {Boolean} loading         - 遮罩层
 * @data {Array}   ids             - 选中数组
 * @data {Boolean} single          - 非单个禁用
 * @data {Boolean} multiple        - 非多个禁用
 * @data {Boolean} showSearch      - 显示搜索条件
 * @data {Number}  total           - 总条数
 * @data {Array}   jobList         - 定时任务表格数据
 * @data {String}  title           - 弹出层标题
 * @data {Boolean} open            - 是否显示弹出层
 * @data {Boolean} openView        - 是否显示详细弹出层
 * @data {Boolean} openCron        - 是否显示Cron表达式弹出层
 * @data {String}  expression      - 传入的表达式
 * @data {Boolean} submitting      - 提交中状态
 * @data {Array}   changingIds     - 状态修改中的任务编号集合
 * @data {Array}   runningIds      - 执行中的任务编号集合
 * @data {Boolean} runtimeOpen     - 是否显示运行记录弹窗
 * @data {Object}  runtimeContext  - 运行记录弹窗上下文
 * @data {Array}   timeZoneOptions - 时区选项
 * @data {Object}  syncStates      - 同步状态字典
 * @data {Object}  queryParams     - 查询参数
 * @data {Object}  form            - 表单参数
 * @data {Object}  rules           - 表单校验规则
 *
 * 事件说明：
 * @event parseJobParameter     - 解析任务参数
 * @event validateParameter     - 校验任务参数
 * @event validateGraceTime     - 校验允许延迟
 * @event handleRuntime         - 打开任务运行记录弹窗
 * @event showMutationResult    - 展示任务变更结果
 * @event stopListRefresh       - 停止列表自动刷新
 * @event handlePageVisibility  - 处理页面显示状态变化
 * @event getList               - 查询定时任务列表
 * @event jobExecutorFormat     - 任务执行器名字典翻译
 * @event cancel                - 取消按钮
 * @event reset                 - 表单重置
 * @event handleQuery           - 搜索按钮操作
 * @event resetQuery            - 重置按钮操作
 * @event handleSelectionChange - 多选框选中数据
 * @event handleCommand         - 更多操作触发
 * @event handleStatusChange    - 任务状态修改
 * @event handleRun             - 立即执行一次
 * @event handleView            - 任务详细信息
 * @event handleShowCron        - cron表达式按钮操作
 * @event crontabFill           - 确定后回传值
 * @event handleJobLog          - 任务日志列表查询
 * @event handleAdd             - 新增按钮操作
 * @event handleUpdate          - 修改按钮操作
 * @event submitForm            - 提交按钮
 * @event handleDelete          - 删除按钮操作
 * @event handleExport          - 导出按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import {
  listJob,
  getJob,
  delJob,
  addJob,
  updateJob,
  runJob,
  changeJobStatus,
} from '@/api/monitor/job'
import JobDetail from './detail'
import Crontab from '@/components/Crontab'
import JobRuntime from './runtime'
import JsonEditor from '@/components/JsonEditor'
import { parseJobParameter, buildJobPayload, syncStates, notifyJobMutation } from '@/utils/job'
import { checkPermi } from '@/utils/permission'

export default
{
  name: 'Job',

  components:
  {
    Crontab,
    JobDetail,
    JobRuntime,
    JsonEditor
  },

  dicts: ['sys_job_status', 'sys_job_executor', 'sys_job_store'],

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
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 定时任务表格数据
      jobList: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // 是否显示详细弹出层
      openView: false,
      // 是否显示Cron表达式弹出层
      openCron: false,
      // 传入的表达式
      expression: '',
      submitting: false,
      changingIds: [],
      runningIds: [],
      runtimeOpen: false,
      runtimeContext: {
        tab: 'executions',
      },
      timeZoneOptions:
        typeof Intl.supportedValuesOf === 'function'
          ? Intl.supportedValuesOf('timeZone')
          : [this.$store.state.user.appTimezone],
      syncStates: syncStates,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        jobName: undefined,
        jobGroup: undefined,
        status: undefined,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        timeZone: [
          {
            required: true,
            message: 'cron时区不能为空',
            trigger: 'blur',
          },
        ],
        misfireGraceTime: [
          {
            validator: this.validateGraceTime,
            trigger: 'change',
          },
        ],
        maxInstances: [
          {
            required: true,
            type: 'number',
            min: 1,
            message: '最大并发数必须为正整数',
            trigger: 'change',
          },
        ],
        jobKwargs: [
          {
            validator: this.validateParameter,
            trigger: 'blur',
          },
        ],
        jobArgs: [
          {
            validator: this.validateParameter,
            trigger: 'blur',
          },
        ],
        jobStore: [
          {
            required: true,
            message: '调度存储不能为空',
            trigger: 'change',
          },
        ],
        jobGroup: [
          {
            required: true,
            whitespace: true,
            message: '业务分组不能为空',
            trigger: 'blur',
          },
        ],
        jobName: [{ required: true, message: '任务名称不能为空', trigger: 'blur' }],
        invokeTarget: [{ required: true, message: '调用目标字符串不能为空', trigger: 'blur' }],
        cronExpression: [{ required: true, message: 'cron执行表达式不能为空', trigger: 'blur' }],
      },
    }
  },

  computed:
  {
    /**
     * 运行记录查询权限
     * @returns {Boolean} 是否有权限
     */
    canQueryRuntime()
    {
      return checkPermi(['monitor:job:query'])
    },

    /**
     * 不限延迟
     * @returns {Boolean} 是否不限延迟
     */
    unlimitedDelay:
    {
      get()
      {
        return this.form.misfireGraceTime === null
      },

      set(value)
      {
        this.form.misfireGraceTime = value ? null : 1
      },
    },
  },

  /**
   * 初始化内部状态
   */
  beforeCreate()
  {
    this._syncRefreshTimer = undefined
    this._listController = undefined
    this._listVersion = 0
    this._pageActive = true
  },

  /**
   * 组件创建后查询列表
   */
  created()
  {
    this.getList()
  },

  /**
   * 组件挂载完成后监听页面可见性变化
   */
  mounted()
  {
    document.addEventListener('visibilitychange', this.handlePageVisibility)
  },

  /**
   * 组件激活时恢复刷新
   */
  activated()
  {
    if (!this._pageActive)
    {
      this._pageActive = true
      this.getList()
    }
  },

  /**
   * 组件失活时暂停刷新
   */
  deactivated()
  {
    this._pageActive = false
    this.stopListRefresh()
  },

  /**
   * 组件销毁前清理定时器与事件监听
   */
  beforeDestroy()
  {
    this._pageActive = false
    this.stopListRefresh()
    document.removeEventListener('visibilitychange', this.handlePageVisibility)
  },

  methods:
  {
    /**
     * 解析任务参数
     * @param {*} value - 参数值
     * @param {String} field - 字段名
     * @returns {*} 解析结果
     */
    parseJobParameter,

    /**
     * 校验任务参数
     * @param {Object} rule - 校验规则
     * @param {*} value - 参数值
     * @param {Function} callback - 校验回调
     */
    validateParameter(rule, value, callback)
    {
      try
      {
        parseJobParameter(value, rule.field)
        callback()
      }
      catch (error)
      {
        callback(error)
      }
    },

    /**
     * 校验允许延迟
     * @param {Object} _rule - 校验规则
     * @param {*} value - 参数值
     * @param {Function} callback - 校验回调
     */
    validateGraceTime(_rule, value, callback)
    {
      const valid = value === null || (Number.isInteger(value) && value >= 1 && value <= 2147483647)
      callback(valid ? undefined : new Error('允许延迟必须为正整数秒'))
    },

    /**
     * 打开任务运行记录弹窗
     * @param {String} tab - 标签页
     * @param {Object} row - 当前行数据
     * @param {String} executionId - 执行编号
     */
    handleRuntime(tab, row, executionId)
    {
      if (!this.canQueryRuntime) return
      this.runtimeContext = {
        tab,
        jobId: row?.jobId,
        executionId,
      }
      this.runtimeOpen = true
    },

    /**
     * 展示任务变更结果
     * @param {Object} response - 接口响应
     */
    showMutationResult(response)
    {
      notifyJobMutation(this.$modal, response)
      if (response.data?.syncStatus === 'failed')
      {
        const affected = response.data.jobs
        this.handleRuntime('sync', affected?.length === 1 ? affected[0] : undefined)
      }
    },

    /**
     * 停止列表自动刷新
     */
    stopListRefresh()
    {
      clearTimeout(this._syncRefreshTimer)
      this._listController?.abort()
      this._listVersion++
      this.loading = false
    },

    /**
     * 处理页面显示状态变化
     */
    handlePageVisibility()
    {
      if (document.hidden) this.stopListRefresh()
      else if (this._pageActive)
        this.getList({
          silent: true,
        })
    },

    /**
     * 查询定时任务列表
     * @param {Object} options - 查询选项
     */
    async getList(options = {})
    {
      const silent = options?.silent === true
      clearTimeout(this._syncRefreshTimer)
      this._listController?.abort()
      this._listController = new AbortController()
      const version = ++this._listVersion
      if (!silent) this.loading = true
      try
      {
        const response = await listJob(
          {
            ...this.queryParams,
          },
          {
            signal: this._listController.signal,
            skipErrorMessage: silent,
          }
        )
        if (version !== this._listVersion) return
        this.jobList = response.rows
        this.total = response.total
      }
      catch
      {
        // 保留当前列表；主动请求的失败信息由请求层显示。
      }
      finally
      {
        if (version === this._listVersion)
        {
          this.loading = false
          if (
            this._pageActive &&
            !document.hidden &&
            this.jobList.some((job) => ['pending', 'failed'].includes(job.syncStatus))
          )
          {
            this._syncRefreshTimer = setTimeout(
              () =>
                this.getList({
                  silent: true,
                }),
              3000
            )
          }
        }
      }
    },

    /**
     * 任务执行器名字典翻译
     * @param {Object} row - 当前行数据
     * @param {Object} column - 列信息
     * @returns {String} 字典标签
     */
    jobExecutorFormat(row, column)
    {
      return this.selectDictLabel(this.dict.type.sys_job_executor, row.jobExecutor)
    },

    /**
     * 取消按钮
     */
    cancel()
    {
      this.open = false
      this.reset()
    },

    /**
     * 表单重置
     */
    reset()
    {
      this.form = {
        jobId: undefined,
        jobName: undefined,
        jobGroup: 'default',
        jobStore: 'default',
        jobExecutor: 'default',
        jobArgs: '[]',
        jobKwargs: '{}',
        invokeTarget: undefined,
        cronExpression: undefined,
        timeZone: this.$store.state.user.appTimezone,
        misfireGraceTime: 1,
        coalesce: false,
        maxInstances: 1,
        status: '1',
      }
      this.resetForm('form')
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
      this.resetForm('queryForm')
      this.handleQuery()
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的数据集合
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.jobId)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },

    /**
     * 更多操作触发
     * @param {String} command - 操作指令
     * @param {Object} row - 当前行数据
     */
    handleCommand(command, row)
    {
      switch (command)
      {
        case 'handleRun':
          this.handleRun(row)
          break
        case 'handleView':
          this.handleView(row)
          break
        case 'handleExecutions':
          this.handleRuntime('executions', row)
          break
        case 'handleSync':
          this.handleRuntime('sync', row)
          break
        case 'handleJobLog':
          this.handleJobLog(row)
          break
        default:
          break
      }
    },

    /**
     * 任务状态修改
     * @param {Object} row - 当前行数据
     */
    async handleStatusChange(row)
    {
      if (this.changingIds.includes(row.jobId)) return
      this.changingIds.push(row.jobId)
      let text = row.status === '0' ? '启用' : '停用'
      try
      {
        await this.$modal.confirm(`确认要${text}任务“${row.jobName}”吗？`)
        const response = await changeJobStatus(row.jobId, row.status)
        this.showMutationResult(response)
        await this.getList()
      }
      catch
      {
        row.status = row.status === '0' ? '1' : '0'
      }
      finally
      {
        this.changingIds = this.changingIds.filter((id) => id !== row.jobId)
      }
    },

    /**
     * 立即执行一次
     * @param {Object} row - 当前行数据
     */
    async handleRun(row)
    {
      if (this.runningIds.includes(row.jobId)) return
      this.runningIds.push(row.jobId)
      try
      {
        await this.$modal.confirm(`确认要立即执行一次任务“${row.jobName}”吗？`)
        const response = await runJob(row.jobId)
        this.$modal.msg(response.msg)
        this.handleRuntime('executions', row, response.data.executionId)
      }
      catch
      {
        // 用户取消或请求层已显示失败原因。
      }
      finally
      {
        this.runningIds = this.runningIds.filter((id) => id !== row.jobId)
      }
    },

    /**
     * 任务详细信息
     * @param {Object} row - 当前行数据
     */
    handleView(row)
    {
      getJob(row.jobId).then((response) =>
      {
        this.form = response.data
        this.openView = true
      })
    },

    /**
     * cron表达式按钮操作
     */
    handleShowCron()
    {
      this.expression = this.form.cronExpression
      this.openCron = true
    },

    /**
     * 确定后回传值
     * @param {String} value - cron 表达式
     */
    crontabFill(value)
    {
      this.form.cronExpression = value
    },

    /**
     * 任务日志列表查询
     * @param {Object} row - 当前行数据
     */
    handleJobLog(row)
    {
      const jobId = row.jobId || 0
      this.$router.push('/monitor/job-log/index/' + jobId)
    },

    /**
     * 新增按钮操作
     */
    handleAdd()
    {
      this.reset()
      this.open = true
      this.title = '添加任务'
    },

    /**
     * 修改按钮操作
     * @param {Object} row - 当前行数据
     */
    handleUpdate(row)
    {
      this.reset()
      const jobId = row.jobId || this.ids
      getJob(jobId).then((response) =>
      {
        this.form = {
          ...response.data,
          jobArgs: JSON.stringify(response.data.jobArgs ?? [], null, 2),
          jobKwargs: JSON.stringify(response.data.jobKwargs ?? {}, null, 2),
        }
        this.open = true
        this.title = '修改任务'
      })
    },

    /**
     * 提交按钮
     */
    submitForm()
    {
      if (this.submitting) return
      this.$refs['form'].validate(async (valid) =>
      {
        if (!valid || this.submitting) return
        this.submitting = true
        try
        {
          const save = this.form.jobId != undefined ? updateJob : addJob
          const response = await save(buildJobPayload(this.form))
          this.open = false
          this.showMutationResult(response)
          this.getList()
        }
        catch
        {
          // 保存失败时保留表单；配置已保存但同步失败时显示同步状态。
        }
        finally
        {
          this.submitting = false
        }
      })
    },

    /**
     * 删除按钮操作
     * @param {Object} row - 当前行数据
     */
    handleDelete(row)
    {
      const jobIds = row.jobId || this.ids
      this.$modal
        .confirm('是否确认删除定时任务编号为"' + jobIds + '"的数据项？')
        .then(function ()
        {
          return delJob(jobIds)
        })
        .then((response) =>
        {
          this.getList()
          this.showMutationResult(response)
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
        'monitor/job/export',
        {
          ...this.queryParams,
        },
        `job_${new Date().getTime()}.xlsx`
      )
    },
  },
}
</script>