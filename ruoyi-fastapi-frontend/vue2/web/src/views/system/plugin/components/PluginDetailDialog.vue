<style lang="scss" scoped>
.mb16 { margin-bottom: 16px; }
.detail-section-title { margin: 14px 0 8px; font-size: 13px; font-weight: 600; color: #606266; }
.detail-section-title-first { margin-top: 0; }
.detail-tag-list { display: flex; flex-wrap: wrap; align-items: center; gap: 6px; min-height: 24px; }
.detail-empty-text { color: #909399; }
.detail-row-actions { display: flex; justify-content: center; gap: 4px; }
</style>

<template>
  <el-dialog
    title="插件详情"
    :visible="visible"
    width="920px"
    append-to-body
    @close="emit('update:visible', false)">
    <el-tabs>
      <el-tab-pane label="概览">
        <div class="detail-section-title detail-section-title-first">基础信息</div>
        <el-descriptions
          class="mb16"
          :column="2"
          border>
          <el-descriptions-item label="插件ID">{{ detail.pluginId }}</el-descriptions-item>
          <el-descriptions-item label="插件名称">{{ detail.pluginName }}</el-descriptions-item>
          <el-descriptions-item label="源码版本">{{ detail.version }}</el-descriptions-item>
          <el-descriptions-item label="已安装版本">{{
            detail.installedVersion || '-'
          }}</el-descriptions-item>
          <el-descriptions-item label="启用设置">{{
            detail.enabled === '0' ? '启用' : '停用'
          }}</el-descriptions-item>
          <el-descriptions-item label="插件状态">{{
            getStatusLabel(detail.status)
          }}</el-descriptions-item>
          <el-descriptions-item label="来源">{{ detail.source || '-' }}</el-descriptions-item>
          <el-descriptions-item label="更新时间">{{
            formatPluginTime(detail.updateTime)
          }}</el-descriptions-item>
          <el-descriptions-item label="分类">{{ metadata.category || '-' }}</el-descriptions-item>
          <el-descriptions-item label="标签">{{
            formatDetailTags(metadata.tags)
          }}</el-descriptions-item>
          <el-descriptions-item label="作者">{{ metadata.author || '-' }}</el-descriptions-item>
          <el-descriptions-item label="许可证">{{ metadata.license || '-' }}</el-descriptions-item>
          <el-descriptions-item
            label="最近错误"
            :span="2">{{ detail.lastError || '-' }}</el-descriptions-item>
          <el-descriptions-item
            label="插件说明"
            :span="2">{{ detail.description || '-' }}</el-descriptions-item>
        </el-descriptions>

        <div class="detail-section-title">后端声明</div>
        <el-descriptions
          class="mb16"
          :column="2"
          border>
          <el-descriptions-item
            label="后端路径"
            :span="2">{{ detail.backendPath || '-' }}</el-descriptions-item>
          <el-descriptions-item
            label="后端模块"
            :span="2">{{ backend.module || '-' }}</el-descriptions-item>
          <el-descriptions-item label="自动扫描路由">{{
            formatYesNo(backend.autoScanRouters)
          }}</el-descriptions-item>
          <el-descriptions-item label="定时任务">{{ backendJobs.length }}</el-descriptions-item>
        </el-descriptions>

        <div class="detail-section-title">前端声明</div>
        <el-descriptions :column="2" border>
          <el-descriptions-item
            label="前端路径"
            :span="2">{{ detail.frontendPath || '-' }}</el-descriptions-item>
          <el-descriptions-item label="前端插件ID">{{
            frontend.pluginId || '-'
          }}</el-descriptions-item>
          <el-descriptions-item label="前端基础路径">{{
            frontend.basePath || '-'
          }}</el-descriptions-item>
          <el-descriptions-item label="视图目录">{{
            frontend.viewsPath || '-'
          }}</el-descriptions-item>
          <el-descriptions-item label="API 目录">{{
            frontend.apiPath || '-'
          }}</el-descriptions-item>
          <el-descriptions-item label="前端交付">{{
            getFrontendDeliveryLabel(frontend.delivery)
          }}</el-descriptions-item>
          <el-descriptions-item label="菜单声明">{{ frontendMenus.length }}</el-descriptions-item>
        </el-descriptions>
      </el-tab-pane>
      <el-tab-pane label="菜单">
        <el-table
          :data="frontendMenus"
          size="small"
          border
          empty-text="暂无菜单声明">
          <el-table-column
            label="名称"
            prop="name"
            min-width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="路径"
            prop="path"
            min-width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="组件"
            prop="component"
            min-width="180"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="权限"
            prop="perms"
            min-width="160"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.perms || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="类型"
            prop="type"
            width="70"
            align="center"/>
          <el-table-column
            label="排序"
            prop="orderNum"
            width="70"
            align="center"/>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="权限">
        <el-table
          :data="detail.permissions || []"
          size="small"
          border
          empty-text="暂无权限声明">
          <el-table-column
            label="权限标识"
            prop="code"
            min-width="180"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="展示名称"
            prop="name"
            min-width="140"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.name || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="description"
            min-width="220"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.description || '-' }}</template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="配置">
        <el-table
          :data="detail.config || []"
          size="small"
          border
          empty-text="暂无配置声明">
          <el-table-column
            label="配置键"
            prop="key"
            min-width="160"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="名称"
            prop="label"
            min-width="140"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.label || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="类型"
            prop="type"
            width="100"
            align="center"/>
          <el-table-column
            label="必填"
            width="70"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.required ? 'warning' : 'info'">{{
                scope.row.required ? '是' : '否'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="敏感"
            width="70"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.secret ? 'warning' : 'info'">{{
                scope.row.secret ? '是' : '否'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="默认值"
            min-width="160"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ formatConfigDefaultValue(scope.row) }}</template>
          </el-table-column>
          <el-table-column
            label="约束"
            min-width="200"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ formatConfigConstraint(scope.row) }}</template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="description"
            min-width="220"
            :show-overflow-tooltip="true"/>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="依赖">
        <div class="detail-section-title detail-section-title-first">运行依赖</div>
        <el-table
          :data="detailDependencyRows"
          size="small"
          border
          empty-text="暂无运行依赖声明">
          <el-table-column
            label="类型"
            prop="label"
            width="120"
            align="center"/>
          <el-table-column
            label="数量"
            width="80"
            align="center">
            <template slot-scope="scope">{{ scope.row.items.length }}</template>
          </el-table-column>
          <el-table-column
            label="依赖项"
            min-width="520">
            <template slot-scope="scope">
              <div class="detail-tag-list">
                <el-tag
                  v-for="item in scope.row.items"
                  :key="scope.row.kind + item"
                  :type="scope.row.tagType"
                  effect="plain">{{ item }}</el-tag>
                <span class="detail-empty-text" v-if="!scope.row.items.length">-</span>
              </div>
            </template>
          </el-table-column>
        </el-table>

        <div class="detail-section-title">数据库脚本</div>
        <el-table
          :data="detailScriptRows"
          size="small"
          border
          empty-text="暂无数据库脚本声明">
          <el-table-column
            label="类型"
            prop="label"
            width="120"
            align="center"/>
          <el-table-column
            label="数量"
            width="80"
            align="center">
            <template slot-scope="scope">{{ scope.row.items.length }}</template>
          </el-table-column>
          <el-table-column
            label="脚本"
            min-width="520">
            <template slot-scope="scope">
              <div class="detail-tag-list">
                <el-tag
                  v-for="item in scope.row.items"
                  :key="scope.row.kind + item"
                  type="info"
                  effect="plain">{{ item }}</el-tag>
                <span class="detail-empty-text" v-if="!scope.row.items.length">-</span>
              </div>
            </template>
          </el-table-column>
        </el-table>

        <div class="detail-section-title">执行历史</div>
        <el-table
          v-loading="migrationLoading"
          :data="migrationHistory"
          size="small"
          border
          empty-text="暂无 migration 执行历史">
          <el-table-column
            label="脚本"
            prop="migrationPath"
            min-width="220"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="版本"
            prop="version"
            width="100"
            align="center">
            <template slot-scope="scope">{{ scope.row.version || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="状态"
            width="120"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="getMigrationStatusTagType(scope.row.status)">
                {{ getMigrationStatusLabel(scope.row.status) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="次数"
            prop="attemptCount"
            width="70"
            align="center">
            <template slot-scope="scope">{{ formatCount(scope.row.attemptCount) }}</template>
          </el-table-column>
          <el-table-column
            label="语句"
            prop="statementCount"
            width="70"
            align="center">
            <template slot-scope="scope">{{ formatCount(scope.row.statementCount) }}</template>
          </el-table-column>
          <el-table-column
            label="开始时间"
            width="160"
            align="center">
            <template slot-scope="scope">{{
              formatPluginTime(scope.row.startedTime || scope.row.createTime)
            }}</template>
          </el-table-column>
          <el-table-column
            label="结束时间"
            width="160"
            align="center">
            <template slot-scope="scope">{{ formatPluginTime(scope.row.finishedTime) }}</template>
          </el-table-column>
          <el-table-column
            label="错误信息"
            min-width="220"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.errorMessage || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="操作"
            width="110"
            align="center"
            fixed="right">
            <template slot-scope="scope">
              <div class="detail-row-actions">
                <el-tooltip
                  v-if="canMarkMigrationSuccess(scope.row)"
                  content="标记成功"
                  placement="top">
                  <el-button
                    type="text"
                    icon="el-icon-circle-check"
                    v-hasPermi="['system:plugin:edit']"
                    @click="emit('mark-migration-success', scope.row)"/>
                </el-tooltip>
                <el-tooltip
                  v-if="canMarkMigrationFailed(scope.row)"
                  content="标记失败"
                  placement="top">
                  <el-button
                    type="text"
                    icon="el-icon-circle-close"
                    v-hasPermi="['system:plugin:edit']"
                    @click="emit('mark-migration-failed', scope.row)"/>
                </el-tooltip>
                <span
                  v-if="!canMarkMigrationSuccess(scope.row) && !canMarkMigrationFailed(scope.row)">-</span>
              </div>
            </template>
          </el-table-column>
        </el-table>

        <div class="detail-section-title">插件依赖</div>
        <el-table
          :data="detail.pluginDependencies || []"
          size="small"
          border
          empty-text="暂无插件依赖">
          <el-table-column
            label="插件ID"
            prop="id"
            width="160"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="版本约束"
            prop="version"
            width="140"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.version || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="description"
            min-width="220"
            :show-overflow-tooltip="true"/>
        </el-table>
        <div class="detail-section-title">定时任务</div>
        <el-table
          :data="backendJobs"
          size="small"
          border
          empty-text="暂无定时任务声明">
          <el-table-column
            label="任务ID"
            prop="id"
            width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="名称"
            prop="name"
            min-width="140"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.name || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="调用目标"
            prop="callable"
            min-width="220"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="Cron"
            prop="cronExpression"
            width="150"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="默认启用"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.enabled ? 'success' : 'info'">{{
                scope.row.enabled ? '是' : '否'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="description"
            min-width="220"
            :show-overflow-tooltip="true"/>
        </el-table>
      </el-tab-pane>
    </el-tabs>
    <div class="dialog-footer" slot="footer">
      <el-button
        v-if="detail.status === 'error'"
        type="primary"
        v-hasPermi="['system:plugin:edit']"
        @click="emit('repair')">重新安装修复</el-button>
      <el-button @click="emit('update:visible', false)">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script setup name="PluginDetailDialog">
/**
 * 组件名称：PluginDetailDialog
 * 功能说明：插件详情弹窗，展示概览、菜单、权限、配置与依赖信息
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}  visible                  - 是否显示弹窗
 * @prop {Object}   detail                    - 插件详情数据
 * @prop {Function} getStatusLabel            - 获取状态标签
 * @prop {Function} formatPluginTime          - 格式化插件时间
 * @prop {Function} formatConfigDefaultValue  - 格式化默认值
 * @prop {Function} formatConfigConstraint    - 格式化约束
 * @prop {Array}    migrationHistory          - migration 执行历史
 * @prop {Boolean}  migrationLoading          - migration 加载状态
 *
 * 变量说明：
 * @data {Object} metadata             - 插件元数据
 * @data {Object} backend              - 后端声明
 * @data {Object} frontend             - 前端声明
 * @data {Array}  frontendMenus        - 前端菜单声明
 * @data {Array}  backendJobs          - 定时任务声明
 * @data {Array}  detailDependencyRows - 运行依赖行数据
 * @data {Array}  detailScriptRows     - 数据库脚本行数据
 *
 * 事件说明：
 * @event formatYesNo                - 格式化是/否
 * @event formatCount                - 格式化数量
 * @event formatDetailTags           - 格式化标签列表
 * @event getFrontendDeliveryLabel   - 获取前端交付标签
 * @event normalizeDetailItems       - 规范化明细项
 * @event getMigrationStatusLabel    - 获取迁移状态标签
 * @event getMigrationStatusTagType  - 获取迁移状态标签类型
 * @event canMarkMigrationSuccess    - 是否可标记迁移成功
 * @event canMarkMigrationFailed     - 是否可标记迁移失败
 *
 * 插槽说明：
 */

/**
 * 2026/07/28     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { computed } from 'vue'

const props = defineProps({
  visible: { type: Boolean, default: false },
  detail: { type: Object, default: () => ({}) },
  getStatusLabel: { type: Function, required: true },
  formatPluginTime: { type: Function, required: true },
  formatConfigDefaultValue: { type: Function, required: true },
  formatConfigConstraint: { type: Function, required: true },
  migrationHistory: { type: Array, default: () => [] },
  migrationLoading: { type: Boolean, default: false },
})

const emit = defineEmits([
  'update:visible',
  'repair',
  'mark-migration-success',
  'mark-migration-failed',
])

/**
 * 插件元数据
 * @returns {Object} 元数据
 */
const metadata = computed(() => props.detail.metadata || {})

/**
 * 后端声明
 * @returns {Object} 后端声明
 */
const backend = computed(() => props.detail.backend || {})

/**
 * 前端声明
 * @returns {Object} 前端声明
 */
const frontend = computed(() => props.detail.frontend || {})

/**
 * 前端菜单声明
 * @returns {Array} 菜单声明
 */
const frontendMenus = computed(() =>
  Array.isArray(frontend.value.menus) ? frontend.value.menus : []
)

/**
 * 定时任务声明
 * @returns {Array} 定时任务声明
 */
const backendJobs = computed(() => (Array.isArray(backend.value.jobs) ? backend.value.jobs : []))

/**
 * 运行依赖行数据
 * @returns {Array} 运行依赖行数据
 */
const detailDependencyRows = computed(() => {
  const dependencies = props.detail.dependencies || {}
  return [
    {
      kind: 'python',
      label: 'Python',
      tagType: 'success',
      items: normalizeDetailItems(dependencies.python),
    },
    {
      kind: 'npm',
      label: 'npm',
      tagType: 'primary',
      items: normalizeDetailItems(dependencies.npm),
    },
    {
      kind: 'npmDev',
      label: 'npmDev',
      tagType: 'warning',
      items: normalizeDetailItems(dependencies.npmDev),
    },
  ]
})

/**
 * 数据库脚本行数据
 * @returns {Array} 脚本行数据
 */
const detailScriptRows = computed(() => {
  const backend = props.detail.backend || {}
  return [
    {
      kind: 'migration',
      label: 'Migration',
      items: normalizeDetailItems(backend.migrations),
    },
    {
      kind: 'seed',
      label: 'Seed',
      items: normalizeDetailItems(backend.seeds),
    },
  ]
})

/**
 * 格式化是/否
 * @param {*} value - 布尔值
 * @returns {String} 是/否或占位符
 */
function formatYesNo(value)
{
  if (value === undefined || value === null) {
    return '-'
  }
  return value ? '是' : '否'
}

/**
 * 格式化数量
 * @param {Number} value - 数量
 * @returns {Number} 数量或 0
 */
function formatCount(value)
{
  return value === null || value === undefined ? 0 : value
}

/**
 * 格式化标签列表
 * @param {Array} tags - 标签列表
 * @returns {String} 标签文本
 */
function formatDetailTags(tags)
{
  return Array.isArray(tags) && tags.length ? tags.join(', ') : '-'
}

/**
 * 获取前端交付标签
 * @param {Object} delivery - 交付信息
 * @returns {String} 交付标签
 */
function getFrontendDeliveryLabel(delivery)
{
  if (!delivery) {
    return '-'
  }
  const typeMap = {
    none: '无前端资源',
    source: '源码交付',
  }
  return typeMap[delivery.type] || delivery.type || '-'
}

/**
 * 规范化明细项
 * @param {Array} items - 明细项
 * @returns {Array} 过滤后的明细项
 */
function normalizeDetailItems(items)
{
  return Array.isArray(items) ? items.filter(Boolean) : []
}

/**
 * 获取迁移状态标签
 * @param {String} status - 迁移状态
 * @returns {String} 状态标签
 */
function getMigrationStatusLabel(status)
{
  const statusMap = {
    running: '执行中/中断',
    success: '成功',
    failed: '失败',
    unknown: '未知',
  }
  return statusMap[status] || status || '-'
}

/**
 * 获取迁移状态标签类型
 * @param {String} status - 迁移状态
 * @returns {String} 标签类型
 */
function getMigrationStatusTagType(status)
{
  const statusMap = {
    running: 'warning',
    success: 'success',
    failed: 'danger',
    unknown: 'info',
  }
  return statusMap[status] || 'info'
}

/**
 * 是否可标记迁移成功
 * @param {Object} row - 迁移记录
 * @returns {Boolean} 是否可标记
 */
function canMarkMigrationSuccess(row)
{
  return ['running', 'failed'].includes(row?.status)
}

/**
 * 是否可标记迁移失败
 * @param {Object} row - 迁移记录
 * @returns {Boolean} 是否可标记
 */
function canMarkMigrationFailed(row)
{
  return row?.status === 'running'
}
</script>