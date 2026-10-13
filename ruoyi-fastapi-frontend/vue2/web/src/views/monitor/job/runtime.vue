<style lang="scss" scoped>
.runtime-status { margin: 0 0 12px; color: #909399; }
.runtime-alert { margin-bottom: 12px; }
.execution-focus { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 16px; margin-bottom: 12px; }
.execution-details { padding: 4px 24px 12px; overflow-wrap: anywhere; }
.execution-details p { margin: 8px 0; white-space: pre-wrap; }
code { font-family: ui-monospace, Consolas, monospace; overflow-wrap: anywhere; }
</style>

<template>
  <el-dialog
    :visible.sync="dialogVisible"
    title="任务运行记录"
    width="1000px"
    append-to-body>
    <el-tabs v-model="activeTab" @tab-click="resetPage">
      <el-tab-pane label="执行记录" name="executions" />
      <el-tab-pane label="同步状态" name="sync" />
    </el-tabs>
    <el-form
      size="small"
      :inline="true"
      label-width="68px"
      @submit.native.prevent="resetPage">
      <el-form-item label="任务编号">
        <el-input-number
          v-model="queryJobId"
          :min="1"
          :precision="0"
          :controls="false"
          placeholder="全部任务"
          style="width: 240px" />
      </el-form-item>
      <el-form-item>
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-search"
          native-type="submit"
          :loading="loading">查询</el-button>
        <el-button
          size="mini"
          icon="el-icon-refresh"
          :disabled="loading"
          @click="loadRecords()">刷新</el-button>
      </el-form-item>
    </el-form>
    <div class="execution-focus" v-if="focusedExecutionId && activeTab === 'executions'">
      <span>本次执行 ID：<code>{{ focusedExecutionId }}</code></span>
      <el-button
        size="mini"
        type="text"
        @click="showAllExecutions">查看该任务全部记录</el-button>
    </div>
    <el-alert
      class="runtime-alert"
      v-if="loadError"
      :title="loadError"
      type="error"
      :closable="false"
      show-icon />
    <p
      class="runtime-status"
      role="status"
      aria-live="polite"
      aria-atomic="true">
      {{
        loading
          ? '正在获取记录…'
          : `共 ${total} 条记录${processingCount ? `，当前页 ${processingCount} 条处理中` : ''}`
      }}
    </p>
    <el-table
      v-if="activeTab === 'executions'"
      v-loading="loading"
      :data="rows"
      row-key="executionId"
      max-height="460">
      <el-table-column type="expand">
        <template slot-scope="scope">
          <div class="execution-details">
            <p>
              <strong>执行 ID：</strong><code>{{ scope.row.executionId }}</code>
            </p>
            <p><strong>完整结果：</strong>{{ scope.row.message || '尚无执行结果' }}</p>
            <p><strong>提交者：</strong>{{ scope.row.requestedBy || '系统调度' }}</p>
            <p><strong>计划时间：</strong>{{ parseTime(scope.row.scheduledTime) || '等待派发' }}</p>
            <p><strong>结束时间：</strong>{{ parseTime(scope.row.endTime) || '—' }}</p>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="jobId"
        label="任务编号"
        width="90" />
      <el-table-column
        align="center"
        prop="jobName"
        label="任务名称"
        min-width="150"
        show-overflow-tooltip />
      <el-table-column
        align="center"
        label="来源"
        width="75">
        <template slot-scope="scope">{{
          scope.row.source === 'manual' ? '手动' : '定时'
        }}</template>
      </el-table-column>
      <el-table-column
        align="center"
        label="执行状态"
        width="140">
        <template slot-scope="scope">
          <el-tag size="small" :type="executionStates[scope.row.status]?.type || 'info'">
            {{ executionStates[scope.row.status]?.label || scope.row.status }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        label="提交时间"
        width="175">
        <template slot-scope="scope">{{ parseTime(scope.row.createTime) }}</template>
      </el-table-column>
      <el-table-column
        align="center"
        label="耗时"
        width="105">
        <template slot-scope="scope">{{
          scope.row.runDurationMs == null ? '—' : `${scope.row.runDurationMs} 毫秒`
        }}</template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="message"
        label="执行结果"
        min-width="230"
        show-overflow-tooltip />
    </el-table>
    <el-table
      v-else
      v-loading="loading"
      :data="rows"
      row-key="jobId"
      max-height="460">
      <el-table-column
        align="center"
        prop="jobId"
        label="任务编号"
        width="90" />
      <el-table-column
        align="center"
        label="配置类型"
        width="100">
        <template slot-scope="scope">{{ scope.row.deleted ? '删除记录' : '任务配置' }}</template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="configVersion"
        label="最新版本"
        width="95" />
      <el-table-column
        align="center"
        prop="appliedVersion"
        label="已应用版本"
        width="105" />
      <el-table-column
        align="center"
        label="同步状态"
        width="115">
        <template slot-scope="scope">
          <el-tag size="small" :type="syncStates[scope.row.syncStatus]?.type || 'info'">
            {{ syncStates[scope.row.syncStatus]?.label || '待同步' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        label="最近生效时间"
        width="175">
        <template slot-scope="scope">{{ parseTime(scope.row.appliedTime) || '—' }}</template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="syncError"
        label="同步错误"
        min-width="240"
        show-overflow-tooltip />
      <el-table-column
        align="center"
        label="操作"
        width="100">
        <template slot-scope="scope">
          <el-button
            size="mini"
            v-if="scope.row.syncStatus !== 'applied'"
            v-hasPermi="['monitor:job:edit']"
            type="text"
            :loading="retryingId === scope.row.jobId"
            :disabled="retryingId !== null && retryingId !== scope.row.jobId"
            @click="retrySync(scope.row)">重试同步</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="pageNum"
      :limit.sync="pageSize"
      @pagination="loadRecords()" />
    <div class="dialog-footer" slot="footer">
      <el-button @click="dialogVisible = false">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script>
/**
 * 组件名称：JobRuntime
 * 功能说明：任务运行记录弹窗，展示任务执行记录与调度同步状态
 * 研发时间：2026/09/12
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}       visible     - 是否显示弹窗
 * @prop {String}        initialTab  - 初始选中的标签页
 * @prop {Number|String} jobId       - 任务编号
 * @prop {String}        executionId - 执行编号
 *
 * 变量说明：
 * @data {String}  activeTab          - 当前选中的标签页
 * @data {Number}  queryJobId         - 查询的任务编号
 * @data {String}  focusedExecutionId - 聚焦的执行编号
 * @data {Array}   rows               - 表格数据
 * @data {Number}  total              - 总条数
 * @data {Number}  pageNum            - 当前页码
 * @data {Number}  pageSize           - 每页条数
 * @data {Boolean} loading            - 加载状态
 * @data {String}  loadError          - 加载错误信息
 * @data {Number}  retryingId         - 重试中的任务编号
 * @data {Object}  executionStates    - 执行状态字典
 * @data {Object}  syncStates         - 同步状态字典
 *
 * 事件说明：
 * @event stopRequests      - 停止自动刷新并取消未完成的请求
 * @event scheduleRefresh   - 根据执行和同步状态安排下一次刷新
 * @event loadRecords       - 查询任务运行记录
 * @event resetPage         - 重置分页并重新查询
 * @event showAllExecutions - 查看当前任务的全部执行记录
 * @event retrySync         - 重试任务调度同步
 * @event handleVisibility  - 处理页面显示状态变化
 *
 * 插槽说明：
 */

/**
 * 2026/09/12     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { listJobExecutions, listJobSync, retryJobSync } from '@/api/monitor/job'
import { executionStates, syncStates, notifyJobMutation } from '@/utils/job'

export default
{
  name: 'JobRuntime',

  props:
  {
    visible: Boolean,
    initialTab: { type: String, default: 'executions' },
    jobId: { type: [Number, String], default: undefined },
    executionId: { type: String, default: undefined },
  },

  /**
   * 组件数据
   * @returns {Object} 组件响应式数据
   */
  data()
  {
    return {
      activeTab: 'executions',
      queryJobId: undefined,
      focusedExecutionId: undefined,
      rows: [],
      total: 0,
      pageNum: 1,
      pageSize: 10,
      loading: false,
      loadError: '',
      retryingId: null,
      executionStates: executionStates,
      syncStates: syncStates,
    }
  },

  computed:
  {
    /**
     * 弹窗显示状态
     * @returns {Boolean} 是否显示
     */
    dialogVisible:
    {
      /**
       * 读取弹窗显示状态
       * @returns {Boolean} 是否显示
       */
      get()
      {
        return this.visible
      },

      /**
       * 更新弹窗显示状态并向上抛出事件
       * @param {Boolean} value - 新的显示状态
       */
      set(value)
      {
        return this.$emit('update:visible', value)
      },
    },

    /**
     * 处理中记录数量
     * @returns {Number} 记录数量
     */
    processingCount()
    {
      return this.rows.filter((row) =>
        this.activeTab === 'executions'
          ? ['pending', 'submitted', 'running'].includes(row.status)
          : row.syncStatus === 'pending'
      ).length
    },

    /**
     * 运行记录监听参数
     * @returns {Array} 监听参数集合
     */
    runtimeQuery()
    {
      return [this.visible, this.initialTab, this.jobId, this.executionId]
    },
  },

  watch:
  {
    /**
     * 监听运行记录参数变化并重新加载
     */
    runtimeQuery:
    {
      handler()
      {
        this.stopRequests()
        if (!this.visible) return
        this.activeTab = this.initialTab
        this.queryJobId = this.jobId == null ? undefined : Number(this.jobId)
        this.focusedExecutionId = this.executionId
        this.loadError = ''
        this.resetPage()
      },
      immediate: true,
    },
  },

  /**
   * 初始化内部请求状态
   */
  beforeCreate()
  {
    this._timer = undefined
    this._controller = undefined
    this._requestVersion = 0
  },

  /**
   * 组件挂载完成后监听页面可见性变化
   */
  mounted()
  {
    document.addEventListener('visibilitychange', this.handleVisibility)
  },

  /**
   * 组件失活时停止请求并关闭弹窗
   */
  deactivated()
  {
    this.stopRequests()
    this.$emit('update:visible', false)
  },

  /**
   * 组件销毁前清理请求与事件监听
   */
  beforeDestroy()
  {
    this.stopRequests()
    document.removeEventListener('visibilitychange', this.handleVisibility)
  },

  methods:
  {
    /**
     * 停止自动刷新并取消未完成的请求
     */
    stopRequests()
    {
      clearTimeout(this._timer)
      this._controller?.abort()
      this._requestVersion++
      this.loading = false
    },

    /**
     * 根据执行和同步状态安排下一次刷新
     */
    scheduleRefresh()
    {
      clearTimeout(this._timer)
      const needsRefresh =
        this.processingCount > 0 ||
        this.loadError ||
        (this.activeTab === 'sync' && this.rows.some((row) => row.syncStatus === 'failed'))
      if (this.visible && !document.hidden && needsRefresh)
      {
        this._timer = setTimeout(() => this.loadRecords(true), 3000)
      }
    },

    /**
     * 查询任务运行记录
     * @param {Boolean} silent - 是否静默加载
     */
    async loadRecords(silent = false)
    {
      if (!this.visible) return
      clearTimeout(this._timer)
      this._controller?.abort()
      this._controller = new AbortController()
      const version = ++this._requestVersion
      if (!silent) this.loading = true
      const query = {
        jobId: this.queryJobId || undefined,
        pageNum: this.pageNum,
        pageSize: this.pageSize,
      }
      if (this.activeTab === 'executions' && this.focusedExecutionId)
        query.executionId = this.focusedExecutionId
      const fetchRecords = this.activeTab === 'executions' ? listJobExecutions : listJobSync
      try
      {
        const response = await fetchRecords(query, {
          signal: this._controller.signal,
          skipErrorMessage: true,
        })
        if (version !== this._requestVersion || !this.visible) return
        this.rows = response.rows
        this.total = response.total
        this.loadError = ''
      }
      catch (error)
      {
        if (version === this._requestVersion && error.code !== 'ERR_CANCELED')
        {
          this.loadError = '获取记录失败，请刷新重试。'
        }
      }
      finally
      {
        if (version === this._requestVersion)
        {
          this.loading = false
          this.scheduleRefresh()
        }
      }
    },

    /**
     * 重置分页并重新查询
     */
    resetPage()
    {
      this.pageNum = 1
      this.rows = []
      this.total = 0
      this.loadRecords()
    },

    /**
     * 查看当前任务的全部执行记录
     */
    showAllExecutions()
    {
      this.focusedExecutionId = undefined
      this.resetPage()
    },

    /**
     * 重试任务调度同步
     * @param {Object} row - 当前行数据
     */
    async retrySync(row)
    {
      this.retryingId = row.jobId
      try
      {
        const response = await retryJobSync(row.jobId)
        notifyJobMutation(this.$modal, response)
        this.$emit('sync-updated')
        await this.loadRecords(true)
      }
      catch
      {
        // 请求层已显示重试失败原因，保留当前记录。
      }
      finally
      {
        this.retryingId = null
      }
    },

    /**
     * 处理页面显示状态变化
     */
    handleVisibility()
    {
      if (document.hidden) this.stopRequests()
      else if (this.visible) this.loadRecords(true)
    },
  },
}
</script>