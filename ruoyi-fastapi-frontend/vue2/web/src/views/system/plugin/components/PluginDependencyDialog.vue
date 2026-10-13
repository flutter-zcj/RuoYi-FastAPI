<style lang="scss" scoped>
.mb16 { margin-bottom: 16px; }
.dependency-summary { display: grid; grid-template-columns: repeat(5, minmax(0, 1fr)); gap: 10px; margin-bottom: 16px; }
.dependency-summary-item { min-width: 0; padding: 10px 12px; border: 1px solid #ebeef5; background: #f8f9fb; border-radius: 4px; }
.dependency-summary-label { display: block; margin-bottom: 6px; font-size: 12px; color: #909399; }
.dependency-summary-value { font-size: 14px; font-weight: 600; color: #303133; word-break: break-all; }
.dependency-summary-value.is-danger { color: #f56c6c; }
.dependency-policy-messages { display: grid; gap: 4px; }
.dependency-policy-next { margin-top: 6px; color: #606266; }
.dependency-command-cell { display: flex; align-items: center; gap: 8px; }
.dependency-command-text { flex: 1; overflow: hidden; min-width: 0; font-family: monospace; color: #303133; text-overflow: ellipsis; white-space: nowrap; }
@media (max-width: 1200px) {
  .dependency-summary { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
@media (max-width: 768px) {
  .dependency-summary { grid-template-columns: 1fr; }
}
</style>

<template>
  <el-dialog
    title="插件依赖"
    :visible="visible"
    width="860px"
    append-to-body
    @close="emit('update:visible', false)">
    <el-alert
      class="mb16"
      v-if="result.message"
      :title="result.message"
      :type="result.ok ? 'success' : 'warning'"
      show-icon
      :closable="false"/>
    <div class="dependency-summary">
      <div class="dependency-summary-item">
        <span class="dependency-summary-label">插件ID</span>
        <span class="dependency-summary-value">{{ result.pluginId || '-' }}</span>
      </div>
      <div class="dependency-summary-item">
        <span class="dependency-summary-label">依赖检查</span>
        <el-tag :type="result.dependencyOk ? 'success' : 'danger'">{{
          formatBoolean(result.dependencyOk)
        }}</el-tag>
      </div>
      <div class="dependency-summary-item">
        <span class="dependency-summary-label">缺失依赖</span>
        <span class="dependency-summary-value is-danger">{{ dependencyProblemCount }}</span>
      </div>
      <div class="dependency-summary-item">
        <span class="dependency-summary-label">安装计划</span>
        <span class="dependency-summary-value">{{ formatPlanCount(result) }}</span>
      </div>
      <div class="dependency-summary-item">
        <span class="dependency-summary-label">安装策略</span>
        <el-tag
          v-if="policy"
          :type="policyTagType">{{ policy.mode || '-' }}</el-tag>
        <span class="dependency-summary-value" v-else>-</span>
      </div>
    </div>
    <el-alert
      class="mb16"
      v-if="policy"
      :title="policySummaryTitle"
      :type="policyAllowed ? 'success' : 'warning'"
      show-icon
      :closable="false">
      <div class="dependency-policy-messages" v-if="policyMessages.length">
        <div
          v-for="message in policyMessages"
          :key="message">
          {{ message }}
        </div>
      </div>
      <div class="dependency-policy-next" v-if="policyNextStep">
        {{ policyNextStep }}
      </div>
    </el-alert>
    <el-tabs>
      <el-tab-pane label="依赖状态">
        <el-table
          :data="result.dependencies || []"
          size="small"
          border
          empty-text="暂无依赖声明">
          <el-table-column
            label="类型"
            prop="kind"
            width="90"
            align="center"/>
          <el-table-column
            label="依赖"
            prop="requirement"
            min-width="200"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="已安装版本"
            min-width="130"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.installedVersion || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="安装状态"
            width="100"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="getDependencyTagType(scope.row)">{{
                formatDependencyInstalled(scope.row)
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="版本满足"
            width="100"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="getDependencyTagType(scope.row)">{{
                formatDependencyVersionSatisfied(scope.row)
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="message"
            min-width="240"
            :show-overflow-tooltip="true"/>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="安装计划">
        <el-table
          :data="result.plan || []"
          size="small"
          border
          empty-text="暂无安装计划">
          <el-table-column
            label="类型"
            prop="kind"
            width="90"
            align="center"/>
          <el-table-column
            label="依赖"
            prop="requirement"
            min-width="150"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="工作目录"
            prop="workdir"
            min-width="220"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="命令"
            min-width="300">
            <template slot-scope="scope">
              <div class="dependency-command-cell">
                <span class="dependency-command-text">{{ scope.row.commandText || '-' }}</span>
                <el-tooltip content="复制命令" placement="top">
                  <el-button
                    type="text"
                    icon="el-icon-document-copy"
                    :disabled="!scope.row.commandText"
                    @click="copyDependencyCommand(scope.row.commandText)"/>
                </el-tooltip>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane v-if="policy" label="策略判定">
        <el-descriptions
          class="mb16"
          :column="2"
          border
          size="small">
          <el-descriptions-item label="策略模式">{{ policy.mode || '-' }}</el-descriptions-item>
          <el-descriptions-item label="允许执行">
            <el-tag :type="policyAllowed ? 'success' : 'danger'">{{
              policyAllowed ? '允许' : '阻断'
            }}</el-tag>
          </el-descriptions-item>
          <el-descriptions-item label="阻断原因">{{
            formatPolicyList(policy.reasons)
          }}</el-descriptions-item>
          <el-descriptions-item label="前置要求">{{
            formatPolicyList(policy.requirements)
          }}</el-descriptions-item>
          <el-descriptions-item label="风险提示">{{
            formatPolicyList(policy.warnings)
          }}</el-descriptions-item>
        </el-descriptions>
        <el-table
          :data="policy.items || []"
          size="small"
          border
          empty-text="暂无策略项">
          <el-table-column
            label="类型"
            prop="kind"
            width="90"
            align="center"/>
          <el-table-column
            label="依赖"
            prop="requirement"
            min-width="180"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="判定"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.allowed ? 'success' : 'danger'">{{
                scope.row.allowed ? '允许' : '阻断'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="锁定版本"
            min-width="120"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.lockedVersion || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="离线制品"
            min-width="220"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.artifactPath || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="说明"
            min-width="260"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ formatPolicyItemMessage(scope.row) }}</template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>
    <div class="dialog-footer" slot="footer">
      <el-button
        type="primary"
        :disabled="isCapabilityOperationBlocked(result.capability, 'dependency_install')"
        :loading="loading"
        @click="emit('dry-run')">生成安装计划</el-button>
      <el-button
        type="success"
        :disabled="!canInstall"
        :loading="loading"
        v-hasPermi="['system:plugin:edit']"
        @click="emit('install')">执行依赖安装</el-button>
      <el-button @click="emit('update:visible', false)">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script setup name="PluginDependencyDialog">
/**
 * 组件名称：PluginDependencyDialog
 * 功能说明：插件依赖弹窗，展示依赖状态、安装计划与策略判定
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}  visible                       - 是否显示弹窗
 * @prop {Object}   result                        - 依赖检查结果
 * @prop {Boolean}  loading                       - 加载状态
 * @prop {Function} isCapabilityOperationBlocked  - 判断能力操作是否被阻断
 * @prop {Function} formatBoolean                 - 格式化布尔值
 * @prop {Function} getDependencyTagType          - 获取依赖标签类型
 * @prop {Function} formatDependencyInstalled     - 格式化依赖安装状态
 * @prop {Function} formatDependencyVersionSatisfied - 格式化依赖版本满足情况
 *
 * 变量说明：
 * @data {Number}  dependencyProblemCount - 缺失依赖数量
 * @data {Object}  policy                 - 策略数据
 * @data {Boolean} policyAllowed          - 策略是否允许执行
 * @data {String}  policyTagType          - 策略标签类型
 * @data {String}  policySummaryTitle     - 策略摘要标题
 * @data {Array}   policyMessages         - 策略消息列表
 * @data {String}  policyNextStep         - 策略下一步提示
 * @data {Boolean} canInstall             - 是否可执行依赖安装
 *
 * 事件说明：
 * @event formatPolicyList       - 格式化策略列表
 * @event formatPlanCount        - 格式化安装计划数量
 * @event formatPolicyItemMessage - 格式化策略项说明
 * @event copyDependencyCommand  - 复制依赖安装命令
 * @event copyText               - 复制文本到剪贴板
 *
 * 插槽说明：
 */

/**
 * 2026/07/28     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { computed } from 'vue'
import { Message } from 'element-ui'

const props = defineProps({
  visible: { type: Boolean, default: false },
  result: { type: Object, default: () => ({}) },
  loading: { type: Boolean, default: false },
  isCapabilityOperationBlocked: { type: Function, required: true },
  formatBoolean: { type: Function, required: true },
  getDependencyTagType: { type: Function, required: true },
  formatDependencyInstalled: { type: Function, required: true },
  formatDependencyVersionSatisfied: { type: Function, required: true },
})

const emit = defineEmits(['update:visible', 'dry-run', 'install'])

/**
 * 缺失依赖数量
 * @returns {Number} 缺失依赖数量
 */
const dependencyProblemCount = computed(() => {
  const dependencies = Array.isArray(props.result.dependencies) ? props.result.dependencies : []
  return dependencies.filter((item) => item.status !== 'skipped' && !item.ok).length
})

/**
 * 策略数据
 * @returns {Object|null} 策略数据
 */
const policy = computed(() => {
  const policyPayload = props.result.policy
  return policyPayload && typeof policyPayload === 'object' ? policyPayload : null
})

/**
 * 策略是否允许执行
 * @returns {Boolean} 是否允许
 */
const policyAllowed = computed(() => policy.value?.allowed === true)

/**
 * 策略标签类型
 * @returns {String} 标签类型
 */
const policyTagType = computed(() => (policyAllowed.value ? 'success' : 'warning'))

/**
 * 策略摘要标题
 * @returns {String} 摘要标题
 */
const policySummaryTitle = computed(() =>
  policyAllowed.value ? '策略允许执行真实安装' : '策略已阻断真实安装'
)

/**
 * 策略消息列表
 * @returns {Array} 策略消息
 */
const policyMessages = computed(() => {
  if (!policy.value) {
    return []
  }
  return [
    ...(Array.isArray(policy.value.reasons) ? policy.value.reasons : []),
    ...(Array.isArray(policy.value.requirements) ? policy.value.requirements : []),
    ...(Array.isArray(policy.value.warnings) ? policy.value.warnings : []),
  ]
})

/**
 * 策略下一步提示
 * @returns {String} 下一步提示
 */
const policyNextStep = computed(() => {
  if (!policy.value || policyAllowed.value) {
    return ''
  }
  if (policy.value.mode === 'plan_only') {
    return 'Web 管理端仅生成安装计划；真实安装请使用 CLI install-deps 受控执行。'
  }
  return '请按策略要求补齐确认参数、锁文件或离线制品后重试。'
})

/**
 * 是否可执行依赖安装
 * @returns {Boolean} 是否可安装
 */
const canInstall = computed(() => {
  return (
    !props.isCapabilityOperationBlocked(props.result.capability, 'dependency_install') &&
    Array.isArray(props.result.plan) &&
    props.result.plan.length > 0 &&
    policy.value?.allowed !== false
  )
})

/**
 * 格式化策略列表
 * @param {Array} items - 策略项列表
 * @returns {String} 拼接后的策略文本
 */
function formatPolicyList(items)
{
  return Array.isArray(items) && items.length ? items.join('；') : '-'
}

/**
 * 格式化安装计划数量
 * @param {Object} result - 依赖检查结果
 * @returns {Number} 安装计划数量
 */
function formatPlanCount(result)
{
  return result.planCount === null || result.planCount === undefined
    ? Array.isArray(result.plan)
      ? result.plan.length
      : 0
    : result.planCount
}

/**
 * 格式化策略项说明
 * @param {Object} item - 策略项
 * @returns {String} 策略项说明
 */
function formatPolicyItemMessage(item)
{
  return formatPolicyList([
    ...(Array.isArray(item.reasons) ? item.reasons : []),
    ...(Array.isArray(item.requirements) ? item.requirements : []),
    ...(Array.isArray(item.warnings) ? item.warnings : []),
  ])
}

/**
 * 复制依赖安装命令
 * @param {String} commandText - 命令文本
 */
function copyDependencyCommand(commandText)
{
  if (!commandText) {
    return
  }
  copyText(commandText)
    .then(() => {
      Message.success('命令已复制')
    })
    .catch(() => {
      Message.error('复制失败，请手动复制命令')
    })
}

/**
 * 复制文本到剪贴板
 * @param {String} text - 待复制文本
 * @returns {Promise<void>}
 */
async function copyText(text)
{
  if (navigator.clipboard?.writeText) {
    await navigator.clipboard.writeText(text)
    return
  }
  const textarea = document.createElement('textarea')
  textarea.value = text
  textarea.setAttribute('readonly', 'readonly')
  textarea.style.position = 'fixed'
  textarea.style.left = '-9999px'
  document.body.appendChild(textarea)
  textarea.select()
  const ok = document.execCommand('copy')
  document.body.removeChild(textarea)
  if (!ok) {
    throw new Error('copy failed')
  }
}
</script>