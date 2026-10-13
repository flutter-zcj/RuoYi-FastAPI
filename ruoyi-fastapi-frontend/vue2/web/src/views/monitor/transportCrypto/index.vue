<style lang="scss" scoped>
.mb16 { margin-bottom: 16px; }
.summary-card { height: 140px; }
.summary-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 16px; }
.summary-title { color: #606266; font-size: 14px; }
.summary-value { color: #303133; font-size: 30px; font-weight: 700; line-height: 1.2; }
.summary-desc { margin-top: 10px; color: #909399; font-size: 13px; }
.card-header { display: flex; align-items: center; justify-content: space-between; gap: 12px; }
.card-actions { display: flex; align-items: center; gap: 12px; }
.header-switch { display: inline-flex; align-items: center; gap: 8px; }
.header-switch-label { display: inline-flex; align-items: center; height: 24px; padding: 0 8px; border: 1px solid #d9ecff; border-radius: 999px; color: #409eff; font-size: 12px; line-height: 1; background: #ecf5ff; white-space: nowrap; }
.compact-actions { justify-content: flex-end; }
.card-subtitle { color: #909399; font-size: 12px; }
.tag-list { display: flex; flex-wrap: wrap; gap: 8px; }
.tag-item { margin: 0; }
.health-card { height: 100%; }
.health-item { margin-bottom: 18px; }
.health-item:last-of-type { margin-bottom: 24px; }
.health-label { display: flex; align-items: center; justify-content: space-between; margin-bottom: 8px; color: #606266; font-size: 13px; }
.chart-panel { width: 100%; height: 300px; margin-bottom: 16px; }
.chart-empty { display: flex; align-items: center; justify-content: center; height: 180px; margin-bottom: 16px; color: #909399; font-size: 13px; background: #fafafa; border: 1px dashed #ebeef5; border-radius: 4px; }
.table-title { margin-bottom: 12px; color: #909399; font-size: 12px; font-weight: 500; }
::v-deep .selected-table-row { background: #ecf5ff !important; }
@media (max-width: 992px) {
  .summary-card { height: auto; }
}
@media (max-width: 768px) {
  .card-header { align-items: flex-start; flex-direction: column; }
  .card-actions { width: 100%; justify-content: space-between; }
  .compact-actions { justify-content: space-between; }
  .chart-panel { height: 260px; }
}
</style>

<template>
  <div class="app-container transport-crypto-monitor" v-loading="loading">
    <el-row class="mb16" :gutter="16">
      <el-col
        :xs="24"
        :sm="12"
        :lg="6">
        <el-card class="summary-card" shadow="hover">
          <div class="summary-header">
            <span class="summary-title">传输加密状态</span>
            <el-tag :type="monitorData.transportCryptoEnabled ? 'success' : 'info'">
              {{ monitorData.transportCryptoEnabled ? '已启用' : '未启用' }}
            </el-tag>
          </div>
          <div class="summary-value">{{ modeLabel }}</div>
          <div class="summary-desc">当前模式：{{ monitorData.transportCryptoMode || '-' }}</div>
        </el-card>
      </el-col>

      <el-col
        :xs="24"
        :sm="12"
        :lg="6">
        <el-card class="summary-card" shadow="hover">
          <div class="summary-header">
            <span class="summary-title">请求总览</span>
            <el-tag type="primary">命中规则</el-tag>
          </div>
          <div class="summary-value">{{ formatCount(monitorData.requestsTotal) }}</div>
          <div class="summary-desc">
            明文 {{ formatCount(monitorData.plainRequestsTotal) }} / 加密
            {{ formatCount(monitorData.encryptedRequestsTotal) }}
          </div>
        </el-card>
      </el-col>

      <el-col
        :xs="24"
        :sm="12"
        :lg="6">
        <el-card class="summary-card" shadow="hover">
          <div class="summary-header">
            <span class="summary-title">解密成功率</span>
            <el-tag :type="decryptRateTagType"> {{ decryptSuccessRate.toFixed(1) }}% </el-tag>
          </div>
          <div class="summary-value">{{ formatCount(monitorData.decryptSuccessTotal) }}</div>
          <div class="summary-desc">
            失败 {{ formatCount(monitorData.decryptFailureTotal) }} / 强制拒绝
            {{ formatCount(monitorData.requiredRejectedTotal) }}
          </div>
        </el-card>
      </el-col>

      <el-col
        :xs="24"
        :sm="12"
        :lg="6">
        <el-card class="summary-card" shadow="hover">
          <div class="summary-header">
            <span class="summary-title">响应加密</span>
            <el-tag type="warning">JSON 响应</el-tag>
          </div>
          <div class="summary-value">{{ formatCount(monitorData.encryptedResponsesTotal) }}</div>
          <div class="summary-desc">
            明文 {{ formatCount(monitorData.plainResponsesTotal) }} / 错误加密
            {{ formatCount(monitorData.encryptedErrorResponsesTotal) }}
          </div>
        </el-card>
      </el-col>
    </el-row>

    <el-row class="mb16" :gutter="16">
      <el-col
        :xs="24"
        :lg="16">
        <el-card shadow="never">
          <div class="card-header" slot="header">
            <span>当前配置</span>
            <div class="card-actions">
              <div class="header-switch">
                <span class="header-switch-label">{{ autoRefresh ? '自动刷新' : '手动' }}</span>
                <el-switch
                  v-model="autoRefresh"
                  active-color="#409eff"
                  inactive-color="#c0c4cc"/>
              </div>
              <el-button
                type="primary"
                icon="el-icon-refresh"
                size="small"
                :loading="loading"
                @click="loadMonitorData(true)">刷新</el-button>
            </div>
          </div>

          <el-descriptions :column="2" border>
            <el-descriptions-item label="统计范围">
              {{ monitorScopeLabel }}
            </el-descriptions-item>
            <el-descriptions-item label="应用环境">
              {{ monitorData.appEnv || '-' }}
            </el-descriptions-item>
            <el-descriptions-item label="当前密钥版本">
              <el-tag>{{ monitorData.currentKid || '-' }}</el-tag>
            </el-descriptions-item>
            <el-descriptions-item label="监控起始时间">
              {{ formatMonitorTime(monitorData.startedAt) || '-' }}
            </el-descriptions-item>
            <el-descriptions-item label="支持的密钥版本">
              <div class="tag-list">
                <el-tag
                  v-for="kid in monitorData.supportedKids || []"
                  :key="kid"
                  effect="plain"
                  class="tag-item">
                  {{ kid }}
                </el-tag>
                <span v-if="!(monitorData.supportedKids || []).length">-</span>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="失败原因种类">
              {{ failureReasonRows.length }}
            </el-descriptions-item>
            <el-descriptions-item label="聚合说明" :span="2">
              {{ monitorScopeDescription }}
            </el-descriptions-item>
            <el-descriptions-item label="启用路径">
              <div class="tag-list">
                <el-tag
                  v-for="path in monitorData.enabledPaths || []"
                  :key="path"
                  type="success"
                  effect="plain"
                  class="tag-item">
                  {{ path }}
                </el-tag>
                <span v-if="!(monitorData.enabledPaths || []).length">全部接口</span>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="强制加密路径">
              <div class="tag-list">
                <el-tag
                  v-for="path in monitorData.requiredPaths || []"
                  :key="path"
                  type="warning"
                  effect="plain"
                  class="tag-item">
                  {{ path }}
                </el-tag>
                <span v-if="!(monitorData.requiredPaths || []).length">-</span>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="排除路径" :span="2">
              <div class="tag-list">
                <el-tag
                  v-for="path in monitorData.excludePaths || []"
                  :key="path"
                  type="info"
                  effect="plain"
                  class="tag-item">
                  {{ path }}
                </el-tag>
                <span v-if="!(monitorData.excludePaths || []).length">-</span>
              </div>
            </el-descriptions-item>
          </el-descriptions>
        </el-card>
      </el-col>

      <el-col
        :xs="24"
        :lg="8">
        <el-card class="health-card" shadow="never">
          <div class="card-header" slot="header">
            <span>运行健康度</span>
            <el-tag :type="healthTagLabelType">{{ healthLabel }}</el-tag>
          </div>

          <div class="health-item">
            <div class="health-label">
              <span>解密成功率</span>
              <span>{{ decryptSuccessRate.toFixed(1) }}%</span>
            </div>
            <el-progress :percentage="Number(decryptSuccessRate.toFixed(1))" :status="healthProgressStatus"/>
          </div>

          <div class="health-item">
            <div class="health-label">
              <span>加密请求占比</span>
              <span>{{ encryptedRequestRate.toFixed(1) }}%</span>
            </div>
            <el-progress :percentage="Number(encryptedRequestRate.toFixed(1))" status="success"/>
          </div>

          <div class="health-item">
            <div class="health-label">
              <span>加密响应占比</span>
              <span>{{ encryptedResponseRate.toFixed(1) }}%</span>
            </div>
            <el-progress :percentage="Number(encryptedResponseRate.toFixed(1))"/>
          </div>

          <el-alert
            :title="healthMessage"
            :type="healthTagType"
            :closable="false"
            show-icon/>
        </el-card>
      </el-col>
    </el-row>

    <el-row class="mb16" :gutter="16">
      <el-col
        :xs="24"
        :lg="10">
        <el-card shadow="never">
          <div class="card-header" slot="header">
            <span>失败原因统计</span>
            <div class="card-actions compact-actions">
              <span class="card-subtitle">按 Redis 聚合口径统计</span>
              <el-tag
                v-if="selectedFailureReason"
                type="danger"
                effect="plain"
                closable
                @close="clearFailureReasonSelection">
                {{ selectedFailureReason }}
              </el-tag>
            </div>
          </div>

          <div
            class="chart-panel"
            v-if="failureReasonRows.length"
            ref="failureReasonChartRef"/>
          <div class="chart-empty" v-else>
            暂无失败记录
          </div>

          <div class="table-title">明细数据</div>
          <el-table
            :data="displayedFailureReasonRows"
            empty-text="暂无失败记录"
            max-height="260"
            :row-class-name="getFailureReasonRowClassName"
            @row-click="handleFailureReasonRowClick">
            <el-table-column
              label="失败原因"
              prop="reason"
              min-width="180">
              <template slot-scope="scope">
                <el-tag :type="getFailureTagType(scope.row.reason)" effect="plain">
                  {{ scope.row.reason }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column
              label="次数"
              prop="count"
              width="100"
              align="center"/>
            <el-table-column
              label="占比"
              width="120"
              align="center">
              <template slot-scope="scope">
                {{ formatPercent(scope.row.count, totalFailureReasonCount) }}
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>

      <el-col
        :xs="24"
        :lg="14">
        <el-card shadow="never">
          <div class="card-header" slot="header">
            <span>密钥版本统计</span>
            <div class="card-actions compact-actions">
              <span class="card-subtitle">观察不同 kid 的运行状态</span>
              <el-tag
                v-if="selectedKid"
                type="success"
                effect="plain"
                closable
                @close="clearKidSelection">
                {{ selectedKid }}
              </el-tag>
            </div>
          </div>

          <div
            class="chart-panel"
            v-if="kidStatRows.length"
            ref="kidStatsChartRef"/>
          <div class="chart-empty" v-else>
            暂无密钥统计数据
          </div>

          <div class="table-title">明细数据</div>
          <el-table
            :data="displayedKidStatRows"
            empty-text="暂无数据"
            max-height="260"
            :row-class-name="getKidStatRowClassName"
            @row-click="handleKidStatRowClick">
            <el-table-column
              label="密钥版本"
              prop="kid"
              min-width="140">
              <template slot-scope="scope">
                <el-tag :type="scope.row.kid === monitorData.currentKid ? 'success' : 'info'">
                  {{ scope.row.kid || '-' }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column
              label="加密请求"
              prop="encryptedRequests"
              min-width="110"
              align="center"/>
            <el-table-column
              label="解密成功"
              prop="decryptSuccess"
              min-width="110"
              align="center"/>
            <el-table-column
              label="解密失败"
              prop="decryptFailure"
              min-width="110"
              align="center"/>
            <el-table-column
              label="加密响应"
              prop="encryptedResponses"
              min-width="110"
              align="center"/>
            <el-table-column
              label="成功率"
              min-width="120"
              align="center">
              <template slot-scope="scope">
                {{
                  formatRate(
                    scope.row.decryptSuccess,
                    scope.row.decryptSuccess + scope.row.decryptFailure
                  )
                }}
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>

    <el-card shadow="never">
      <div class="card-header" slot="header">
        <span>最近失败记录</span>
        <div class="card-actions compact-actions">
          <span class="card-subtitle">最近 {{ displayedRecentFailures.length }} 条</span>
          <el-tag
            v-if="selectedFailureReason || selectedKid"
            type="info"
            effect="plain">
            已按当前选择联动筛选
          </el-tag>
        </div>
      </div>

      <el-table :data="displayedRecentFailures" empty-text="暂无失败记录">
        <el-table-column label="时间" min-width="170">
          <template slot-scope="scope">
            {{ formatMonitorTime(scope.row.time) || '-' }}
          </template>
        </el-table-column>
        <el-table-column
          label="请求方法"
          prop="method"
          width="100"
          align="center">
          <template slot-scope="scope">
            <el-tag effect="plain">{{ scope.row.method || '-' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="请求路径"
          prop="path"
          min-width="260"
          show-overflow-tooltip/>
        <el-table-column
          label="失败原因"
          prop="reason"
          min-width="180">
          <template slot-scope="scope">
            <el-tag :type="getFailureTagType(scope.row.reason)" effect="plain">
              {{ scope.row.reason || '-' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="密钥版本"
          prop="kid"
          min-width="140">
          <template slot-scope="scope">
            {{ scope.row.kid || '-' }}
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
/**
 * 组件名称：TransportCryptoMonitorVue2
 * 功能说明：传输加密监控页面，展示加密模式、请求与响应统计、解密成功率、失败原因与密钥版本运行状态
 * 研发时间：2026/04/16
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} loading                  - 加载状态
 * @data {Boolean} autoRefresh              - 是否自动刷新
 * @data {Object}  monitorData              - 监控数据
 * @data {Object}  refreshTimer             - 自动刷新定时器
 * @data {String}  selectedFailureReason    - 选中的失败原因
 * @data {String}  selectedKid              - 选中的密钥版本
 * @data {Object}  failureReasonChartInstance - 失败原因图表实例
 * @data {Object}  kidStatsChartInstance     - 密钥版本图表实例
 *
 * 事件说明：
 * @event formatMonitorTime              - 格式化监控时间
 * @event loadMonitorData                - 加载传输加密监控数据
 * @event resetRefreshTimer              - 重置自动刷新定时器
 * @event formatCount                    - 数值格式化
 * @event formatPercent                  - 计算占比并格式化为百分比
 * @event formatRate                     - 计算比率并格式化为百分比
 * @event getRate                        - 计算比率
 * @event getFailureTagType              - 根据失败原因返回标签类型
 * @event getFailureChartColor           - 根据失败原因返回图表颜色
 * @event buildKidChartBarData           - 构建密钥版本图表柱状数据
 * @event bindFailureReasonChartEvents   - 绑定失败原因图表点击事件
 * @event bindKidStatsChartEvents        - 绑定密钥版本图表点击事件
 * @event initFailureReasonChart         - 初始化失败原因图表实例
 * @event initKidStatsChart              - 初始化密钥版本图表实例
 * @event renderFailureReasonChart       - 渲染失败原因图表
 * @event renderKidStatsChart            - 渲染密钥版本图表
 * @event resizeCharts                   - 调整图表尺寸
 * @event destroyCharts                  - 销毁图表实例
 * @event clearFailureReasonSelection    - 清空失败原因选择
 * @event clearKidSelection              - 清空密钥版本选择
 * @event handleFailureReasonRowClick    - 处理失败原因行点击
 * @event handleKidStatRowClick          - 处理密钥版本行点击
 * @event getFailureReasonRowClassName   - 获取失败原因行样式类
 * @event getKidStatRowClassName         - 获取密钥版本行样式类
 *
 * 插槽说明：
 */

/**
 * 2026/04/16     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getTransportCryptoMonitor } from '@/api/monitor/transportCrypto'
import { parseTime } from '@/utils/ruoyi'
import * as echarts from 'echarts'

const MODE_LABEL_MAP = {
  required: '强制加密',
  optional: '可选加密',
  off: '已关闭',
}

const MONITOR_SCOPE_LABEL_MAP = {
  'redis-aggregated': 'Redis 聚合',
  'redis-aggregated+local-fallback': 'Redis 聚合 + 本地回退',
  'process-local-fallback': '本地回退',
}

/**
 * 创建默认的监控数据结构
 * @returns {Object} 默认监控数据
 */
function createDefaultMonitorData()
{
  return {
    supportedKids: [],
    enabledPaths: [],
    requiredPaths: [],
    excludePaths: [],
    kidStats: [],
    recentFailures: [],
    failureReasons: {},
  }
}

export default
{
  name: 'TransportCryptoMonitorVue2',

  data()
  {
    return {
      loading: false,
      autoRefresh: true,
      monitorData: createDefaultMonitorData(),
      refreshTimer: null,
      selectedFailureReason: '',
      selectedKid: '',
      failureReasonChartInstance: null,
      kidStatsChartInstance: null,
    }
  },

  computed:
  {
    /**
     * 传输加密模式名称
     * @returns {String} 模式名称
     */
    modeLabel()
    {
      return MODE_LABEL_MAP[this.monitorData.transportCryptoMode] || '未配置'
    },

    /**
     * 统计范围名称
     * @returns {String} 统计范围名称
     */
    monitorScopeLabel()
    {
      return (
        MONITOR_SCOPE_LABEL_MAP[this.monitorData.monitorScope] ||
        this.monitorData.monitorScope ||
        '-'
      )
    },

    /**
     * 统计范围说明
     * @returns {String} 统计范围说明
     */
    monitorScopeDescription()
    {
      if (this.monitorData.monitorScope === 'redis-aggregated')
      {
        return '统计结果已聚合到 Redis，可覆盖多 worker / 多实例共享的监控口径。'
      }
      if (this.monitorData.monitorScope === 'redis-aggregated+local-fallback')
      {
        return '当前以 Redis 聚合为主，部分监控写入曾降级到本地内存，建议检查 Redis 连接稳定性。'
      }
      if (this.monitorData.monitorScope === 'process-local-fallback')
      {
        return '当前监控未写入 Redis，页面展示的是本进程本地统计，请优先检查 Redis 可用性。'
      }
      return '当前展示传输加密运行状态与统计信息。'
    },

    /**
     * 失败原因统计行
     * @returns {Array} 按次数降序的失败原因列表
     */
    failureReasonRows()
    {
      const failureReasons = this.monitorData.failureReasons || {}
      return Object.keys(failureReasons)
        .map(function (key) {
          return {
            reason: key,
            count: failureReasons[key],
          }
        })
        .sort(function (a, b) {
          return b.count - a.count
        })
    },

    /**
     * 失败原因总次数
     * @returns {Number} 总次数
     */
    totalFailureReasonCount()
    {
      return this.failureReasonRows.reduce(function (total, item) {
        return total + Number(item.count || 0)
      }, 0)
    },

    /**
     * 按选中失败原因过滤后的统计行
     * @returns {Array} 过滤后的失败原因列表
     */
    displayedFailureReasonRows()
    {
      if (!this.selectedFailureReason)
      {
        return this.failureReasonRows
      }
      return this.failureReasonRows.filter((item) => item.reason === this.selectedFailureReason)
    },

    /**
     * 密钥版本统计行，当前密钥版本置顶
     * @returns {Array} 密钥版本统计列表
     */
    kidStatRows()
    {
      const currentKid = this.monitorData.currentKid
      return (this.monitorData.kidStats || []).slice().sort(function (a, b) {
        if (a.kid === currentKid && b.kid !== currentKid)
        {
          return -1
        }
        if (b.kid === currentKid && a.kid !== currentKid)
        {
          return 1
        }
        const aTotal =
          Number(a.encryptedRequests || 0) +
          Number(a.decryptSuccess || 0) +
          Number(a.decryptFailure || 0)
        const bTotal =
          Number(b.encryptedRequests || 0) +
          Number(b.decryptSuccess || 0) +
          Number(b.decryptFailure || 0)
        return bTotal - aTotal
      })
    },

    /**
     * 按选中密钥版本过滤后的统计行
     * @returns {Array} 过滤后的密钥版本列表
     */
    displayedKidStatRows()
    {
      if (!this.selectedKid)
      {
        return this.kidStatRows
      }
      return this.kidStatRows.filter((item) => item.kid === this.selectedKid)
    },

    /**
     * 按选中条件过滤后的最近失败记录
     * @returns {Array} 过滤后的失败记录
     */
    displayedRecentFailures()
    {
      return (this.monitorData.recentFailures || []).filter((item) => {
        const matchesReason =
          !this.selectedFailureReason || item.reason === this.selectedFailureReason
        const matchesKid = !this.selectedKid || item.kid === this.selectedKid
        return matchesReason && matchesKid
      })
    },

    /**
     * 解密成功率
     * @returns {Number} 解密成功率
     */
    decryptSuccessRate()
    {
      return this.getRate(
        this.monitorData.decryptSuccessTotal,
        this.monitorData.decryptSuccessTotal + this.monitorData.decryptFailureTotal
      )
    },

    /**
     * 加密请求占比
     * @returns {Number} 加密请求占比
     */
    encryptedRequestRate()
    {
      return this.getRate(this.monitorData.encryptedRequestsTotal, this.monitorData.requestsTotal)
    },

    /**
     * 加密响应占比
     * @returns {Number} 加密响应占比
     */
    encryptedResponseRate()
    {
      return this.getRate(
        this.monitorData.encryptedResponsesTotal,
        this.monitorData.encryptedResponsesTotal + this.monitorData.plainResponsesTotal
      )
    },

    /**
     * 解密成功率对应的标签类型
     * @returns {String} 标签类型
     */
    decryptRateTagType()
    {
      if (this.decryptSuccessRate >= 95)
      {
        return 'success'
      }
      if (this.decryptSuccessRate >= 80)
      {
        return 'warning'
      }
      return 'danger'
    },

    /**
     * 运行健康度文本
     * @returns {String} 健康度文本
     */
    healthLabel()
    {
      if ((this.monitorData.decryptFailureTotal || 0) === 0)
      {
        return '稳定'
      }
      if (this.decryptSuccessRate >= 95)
      {
        return '良好'
      }
      if (this.decryptSuccessRate >= 80)
      {
        return '关注'
      }
      return '告警'
    },

    /**
     * 运行健康度标签类型
     * @returns {String} 标签类型
     */
    healthTagType()
    {
      if ((this.monitorData.decryptFailureTotal || 0) === 0)
      {
        return 'success'
      }
      if (this.decryptSuccessRate >= 95)
      {
        return 'success'
      }
      if (this.decryptSuccessRate >= 80)
      {
        return 'warning'
      }
      return 'error'
    },

    /**
     * el-tag 可用的健康度标签类型
     * @returns {String} 标签类型
     */
    healthTagLabelType()
    {
      return this.healthTagType === 'error' ? 'danger' : this.healthTagType
    },

    /**
     * 健康度进度条状态
     * @returns {String} 进度条状态
     */
    healthProgressStatus()
    {
      if (this.decryptSuccessRate >= 95)
      {
        return 'success'
      }
      if (this.decryptSuccessRate >= 80)
      {
        return 'warning'
      }
      return 'exception'
    },

    /**
     * 健康度提示信息
     * @returns {String} 提示信息
     */
    healthMessage()
    {
      if ((this.monitorData.decryptFailureTotal || 0) === 0)
      {
        return '当前暂无解密失败记录，链路运行稳定。'
      }
      if (this.decryptSuccessRate >= 95)
      {
        return '存在少量失败事件，建议继续观察失败原因分布。'
      }
      if (this.decryptSuccessRate >= 80)
      {
        return '近期已有一定比例的异常请求，建议优先检查失败原因与最近失败记录。'
      }
      return '异常比例偏高，建议立即排查密钥版本、AAD 绑定、时间窗与重放校验。'
    },
  },

  watch:
  {
    /**
     * 自动刷新开关变化时重置定时器
     */
    autoRefresh()
    {
      this.resetRefreshTimer()
    },

    /**
     * 选中失败原因变化时刷新失败原因图表
     */
    selectedFailureReason()
    {
      this.$nextTick(() => {
        this.renderFailureReasonChart()
      })
    },

    /**
     * 选中密钥版本变化时刷新密钥版本图表
     */
    selectedKid()
    {
      this.$nextTick(() => {
        this.renderKidStatsChart()
      })
    },
  },

  mounted()
  {
    this.loadMonitorData(true)
    this.resetRefreshTimer()
    window.addEventListener('resize', this.resizeCharts)
  },

  beforeDestroy()
  {
    if (this.refreshTimer)
    {
      clearInterval(this.refreshTimer)
      this.refreshTimer = null
    }
    window.removeEventListener('resize', this.resizeCharts)
    this.destroyCharts()
  },

  methods:
  {
    /**
     * 格式化监控时间
     * @param {*} value - 时间值
     * @param {String} pattern - 时间格式
     * @returns {String} 格式化后的时间
     */
    formatMonitorTime(value, pattern = '{y}-{m}-{d} {h}:{i}:{s}')
    {
      return parseTime(value, pattern)
    },

    /**
     * 加载传输加密监控数据
     * @param {Boolean} showLoading - 是否显示加载状态
     * @returns {Promise} 请求结果
     */
    loadMonitorData(showLoading)
    {
      const needLoading = showLoading !== false
      if (needLoading)
      {
        this.loading = true
      }
      return getTransportCryptoMonitor()
        .then((response) => {
          this.monitorData = Object.assign(createDefaultMonitorData(), response.data || {})
          this.$nextTick(() => {
            this.renderFailureReasonChart()
            this.renderKidStatsChart()
          })
        })
        .catch(() => {
          this.$message.error('加载传输加密监控数据失败')
        })
        .finally(() => {
          if (needLoading) {
            this.loading = false
          }
        })
    },

    /**
     * 重置自动刷新定时器
     */
    resetRefreshTimer()
    {
      if (this.refreshTimer)
      {
        clearInterval(this.refreshTimer)
        this.refreshTimer = null
      }
      if (this.autoRefresh)
      {
        this.refreshTimer = setInterval(() => {
          this.loadMonitorData(false)
        }, 15000)
      }
    },

    /**
     * 数值格式化
     * @param {*} value - 数值
     * @returns {Number} 数值
     */
    formatCount(value)
    {
      return Number(value || 0)
    },

    /**
     * 计算占比并格式化为百分比
     * @param {*} value - 当前值
     * @param {Number} total - 总值
     * @returns {String} 百分比字符串
     */
    formatPercent(value, total)
    {
      if (!total)
      {
        return '0.0%'
      }
      return ((Number(value || 0) / total) * 100).toFixed(1) + '%'
    },

    /**
     * 计算比率并格式化为百分比
     * @param {*} success - 成功值
     * @param {*} total - 总值
     * @returns {String} 百分比字符串
     */
    formatRate(success, total)
    {
      return this.getRate(success, total).toFixed(1) + '%'
    },

    /**
     * 计算比率
     * @param {*} numerator - 分子
     * @param {*} denominator - 分母
     * @returns {Number} 比率
     */
    getRate(numerator, denominator)
    {
      if (!denominator)
      {
        return 0
      }
      return (Number(numerator || 0) / Number(denominator || 0)) * 100
    },

    /**
     * 根据失败原因返回标签类型
     * @param {String} reason - 失败原因
     * @returns {String} 标签类型
     */
    getFailureTagType(reason)
    {
      const warningReasons = ['timestamp_expired', 'required_missing']
      const dangerReasons = ['decrypt_failed', 'aad_mismatch', 'replay_detected', 'kid_mismatch']

      if (dangerReasons.indexOf(reason) !== -1)
      {
        return 'danger'
      }
      if (warningReasons.indexOf(reason) !== -1)
      {
        return 'warning'
      }
      return 'info'
    },

    /**
     * 根据失败原因返回图表颜色
     * @param {String} reason - 失败原因
     * @returns {String} 颜色值
     */
    getFailureChartColor(reason)
    {
      const tagType = this.getFailureTagType(reason)
      if (tagType === 'danger')
      {
        return '#f56c6c'
      }
      if (tagType === 'warning')
      {
        return '#e6a23c'
      }
      return '#409eff'
    },

    /**
     * 构建密钥版本图表柱状数据
     * @param {Object} item - 密钥版本行数据
     * @param {*} value - 数值
     * @param {String} color - 颜色
     * @returns {Object} 柱状数据
     */
    buildKidChartBarData(item, value, color)
    {
      const isSelected = !this.selectedKid || this.selectedKid === item.kid
      return {
        value: Number(value || 0),
        itemStyle: {
          color: color,
          opacity: isSelected ? 1 : 0.3,
        },
      }
    },

    /**
     * 绑定失败原因图表点击事件
     * @param {Object} chartInstance - 图表实例
     */
    bindFailureReasonChartEvents(chartInstance)
    {
      chartInstance.off('click')
      chartInstance.on('click', (params) => {
        const targetReason =
          this.failureReasonRows[params.dataIndex] &&
          this.failureReasonRows[params.dataIndex].reason
        if (!targetReason) {
          return
        }
        this.selectedFailureReason = this.selectedFailureReason === targetReason ? '' : targetReason
      })
    },

    /**
     * 绑定密钥版本图表点击事件
     * @param {Object} chartInstance - 图表实例
     */
    bindKidStatsChartEvents(chartInstance)
    {
      chartInstance.off('click')
      chartInstance.on('click', (params) => {
        const targetKid =
          this.kidStatRows[params.dataIndex] && this.kidStatRows[params.dataIndex].kid
        if (!targetKid) {
          return
        }
        this.selectedKid = this.selectedKid === targetKid ? '' : targetKid
      })
    },

    /**
     * 初始化失败原因图表实例
     * @returns {Object|null} 图表实例
     */
    initFailureReasonChart()
    {
      if (!this.$refs.failureReasonChartRef)
      {
        if (this.failureReasonChartInstance)
        {
          this.failureReasonChartInstance.dispose()
          this.failureReasonChartInstance = null
        }
        return null
      }
      if (!this.failureReasonChartInstance)
      {
        this.failureReasonChartInstance = echarts.init(this.$refs.failureReasonChartRef)
        this.bindFailureReasonChartEvents(this.failureReasonChartInstance)
      }
      return this.failureReasonChartInstance
    },

    /**
     * 初始化密钥版本图表实例
     * @returns {Object|null} 图表实例
     */
    initKidStatsChart()
    {
      if (!this.$refs.kidStatsChartRef)
      {
        if (this.kidStatsChartInstance)
        {
          this.kidStatsChartInstance.dispose()
          this.kidStatsChartInstance = null
        }
        return null
      }
      if (!this.kidStatsChartInstance)
      {
        this.kidStatsChartInstance = echarts.init(this.$refs.kidStatsChartRef)
        this.bindKidStatsChartEvents(this.kidStatsChartInstance)
      }
      return this.kidStatsChartInstance
    },

    /**
     * 渲染失败原因图表
     */
    renderFailureReasonChart()
    {
      if (!this.failureReasonRows.length)
      {
        if (this.failureReasonChartInstance)
        {
          this.failureReasonChartInstance.dispose()
          this.failureReasonChartInstance = null
        }
        return
      }
      const chartInstance = this.initFailureReasonChart()
      if (!chartInstance)
      {
        return
      }
      chartInstance.setOption({
        animationDuration: 400,
        color: this.failureReasonRows.map((item) => this.getFailureChartColor(item.reason)),
        grid: {
          top: 16,
          left: 120,
          right: 24,
          bottom: 16,
          containLabel: true,
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow',
          },
          formatter: (params) => {
            const currentItem = params && params[0]
            if (!currentItem) {
              return ''
            }
            const currentRow = this.failureReasonRows[currentItem.dataIndex]
            return (
              currentRow.reason +
              '<br/>次数：' +
              currentRow.count +
              '<br/>占比：' +
              this.formatPercent(currentRow.count, this.totalFailureReasonCount)
            )
          },
        },
        xAxis: {
          type: 'value',
          splitLine: {
            lineStyle: {
              type: 'dashed',
            },
          },
        },
        yAxis: {
          type: 'category',
          data: this.failureReasonRows.map((item) => item.reason),
          axisTick: {
            show: false,
          },
        },
        series: [
          {
            name: '失败次数',
            type: 'bar',
            barMaxWidth: 22,
            data: this.failureReasonRows.map((item) => ({
              value: Number(item.count || 0),
              itemStyle: {
                color: this.getFailureChartColor(item.reason),
                opacity:
                  !this.selectedFailureReason || this.selectedFailureReason === item.reason
                    ? 1
                    : 0.35,
                borderRadius: [0, 6, 6, 0],
              },
            })),
            label: {
              show: true,
              position: 'right',
            },
          },
        ],
      })
    },

    /**
     * 渲染密钥版本图表
     */
    renderKidStatsChart()
    {
      if (!this.kidStatRows.length)
      {
        if (this.kidStatsChartInstance)
        {
          this.kidStatsChartInstance.dispose()
          this.kidStatsChartInstance = null
        }
        return
      }
      const chartInstance = this.initKidStatsChart()
      if (!chartInstance)
      {
        return
      }
      chartInstance.setOption({
        animationDuration: 400,
        color: ['#409eff', '#67c23a', '#f56c6c', '#909399'],
        legend: {
          top: 0,
        },
        grid: {
          top: 48,
          left: 24,
          right: 24,
          bottom: 32,
          containLabel: true,
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow',
          },
        },
        xAxis: {
          type: 'category',
          axisLabel: {
            interval: 0,
            rotate: this.kidStatRows.length > 4 ? 20 : 0,
          },
          data: this.kidStatRows.map((item) => item.kid || '-'),
        },
        yAxis: {
          type: 'value',
          splitLine: {
            lineStyle: {
              type: 'dashed',
            },
          },
        },
        series: [
          {
            name: '加密请求',
            type: 'bar',
            barMaxWidth: 18,
            data: this.kidStatRows.map((item) =>
              this.buildKidChartBarData(item, item.encryptedRequests, '#409eff')
            ),
          },
          {
            name: '解密成功',
            type: 'bar',
            barMaxWidth: 18,
            data: this.kidStatRows.map((item) =>
              this.buildKidChartBarData(item, item.decryptSuccess, '#67c23a')
            ),
          },
          {
            name: '解密失败',
            type: 'bar',
            barMaxWidth: 18,
            data: this.kidStatRows.map((item) =>
              this.buildKidChartBarData(item, item.decryptFailure, '#f56c6c')
            ),
          },
          {
            name: '加密响应',
            type: 'bar',
            barMaxWidth: 18,
            data: this.kidStatRows.map((item) =>
              this.buildKidChartBarData(item, item.encryptedResponses, '#909399')
            ),
          },
        ],
      })
    },

    /**
     * 调整图表尺寸
     */
    resizeCharts()
    {
      if (this.failureReasonChartInstance)
      {
        this.failureReasonChartInstance.resize()
      }
      if (this.kidStatsChartInstance)
      {
        this.kidStatsChartInstance.resize()
      }
    },

    /**
     * 销毁图表实例
     */
    destroyCharts()
    {
      if (this.failureReasonChartInstance)
      {
        this.failureReasonChartInstance.dispose()
        this.failureReasonChartInstance = null
      }
      if (this.kidStatsChartInstance)
      {
        this.kidStatsChartInstance.dispose()
        this.kidStatsChartInstance = null
      }
    },

    /**
     * 清空失败原因选择
     */
    clearFailureReasonSelection()
    {
      this.selectedFailureReason = ''
    },

    /**
     * 清空密钥版本选择
     */
    clearKidSelection()
    {
      this.selectedKid = ''
    },

    /**
     * 处理失败原因行点击
     * @param {Object} row - 行数据
     */
    handleFailureReasonRowClick(row)
    {
      const targetReason = row && row.reason ? row.reason : ''
      this.selectedFailureReason = this.selectedFailureReason === targetReason ? '' : targetReason
    },

    /**
     * 处理密钥版本行点击
     * @param {Object} row - 行数据
     */
    handleKidStatRowClick(row)
    {
      const targetKid = row && row.kid ? row.kid : ''
      this.selectedKid = this.selectedKid === targetKid ? '' : targetKid
    },

    /**
     * 获取失败原因行样式类
     * @param {Object} params - 表格行参数
     * @returns {String} 样式类名
     */
    getFailureReasonRowClassName(params)
    {
      return this.selectedFailureReason && params.row.reason === this.selectedFailureReason
        ? 'selected-table-row'
        : ''
    },

    /**
     * 获取密钥版本行样式类
     * @param {Object} params - 表格行参数
     * @returns {String} 样式类名
     */
    getKidStatRowClassName(params)
    {
      return this.selectedKid && params.row.kid === this.selectedKid ? 'selected-table-row' : ''
    },
  },
}
</script>