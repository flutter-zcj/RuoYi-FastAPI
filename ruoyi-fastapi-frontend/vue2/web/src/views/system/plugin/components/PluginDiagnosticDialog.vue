<style lang="scss" scoped>
.mb16 { margin-bottom: 16px; }
.diagnostic-summary { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 10px; margin-bottom: 16px; }
.diagnostic-summary-item { min-width: 0; padding: 10px 12px; border: 1px solid #ebeef5; background: #f8f9fb; border-radius: 4px; }
.diagnostic-summary-label { display: block; margin-bottom: 6px; font-size: 12px; color: #909399; }
.diagnostic-summary-value { font-size: 14px; font-weight: 600; color: #303133; word-break: break-all; }
</style>

<template>
  <el-dialog
    :title="title"
    :visible="visible"
    width="920px"
    append-to-body
    @close="emit('update:visible', false)">
    <el-alert
      class="mb16"
      v-if="result.message"
      :title="result.message"
      :type="result.ok ? 'success' : 'warning'"
      show-icon
      :closable="false"/>

    <el-skeleton
      v-if="loading"
      :rows="6"
      animated/>
    <template v-else>
      <div class="diagnostic-summary">
        <div class="diagnostic-summary-item">
          <span class="diagnostic-summary-label">插件ID</span>
          <span class="diagnostic-summary-value">{{ result.pluginId || '-' }}</span>
        </div>
        <div class="diagnostic-summary-item">
          <span class="diagnostic-summary-label">结果</span>
          <el-tag :type="result.ok ? 'success' : 'danger'">{{
            result.ok ? '正常' : '异常'
          }}</el-tag>
        </div>
        <div class="diagnostic-summary-item" v-if="health">
          <span class="diagnostic-summary-label">状态</span>
          <span class="diagnostic-summary-value">{{ health.status || '-' }}</span>
        </div>
        <div class="diagnostic-summary-item" v-if="health">
          <span class="diagnostic-summary-label">耗时</span>
          <span class="diagnostic-summary-value">{{ formatDuration(health.durationMs) }}ms</span>
        </div>
      </div>

      <el-tabs>
        <el-tab-pane v-if="health" label="健康状态">
          <el-descriptions
            class="mb16"
            :column="2"
            border>
            <el-descriptions-item label="检查器">{{ health.checker || '-' }}</el-descriptions-item>
            <el-descriptions-item label="状态">{{ health.status || '-' }}</el-descriptions-item>
            <el-descriptions-item
              label="说明"
              :span="2">{{ health.message || '-' }}</el-descriptions-item>
            <el-descriptions-item
              label="错误"
              :span="2">{{ health.error || '-' }}</el-descriptions-item>
          </el-descriptions>
          <el-input
            :value="formatJson(health.details)"
            type="textarea"
            :rows="8"
            readonly/>
        </el-tab-pane>

        <el-tab-pane v-if="diagnose" label="检查摘要">
          <el-descriptions
            class="mb16"
            :column="3"
            border>
            <el-descriptions-item label="依赖">{{
              formatBoolean(diagnoseCheck.dependencyOk)
            }}</el-descriptions-item>
            <el-descriptions-item label="Manifest">{{
              formatBoolean(diagnoseCheck.manifestOk)
            }}</el-descriptions-item>
            <el-descriptions-item label="结构">{{
              formatBoolean(diagnoseCheck.structureOk)
            }}</el-descriptions-item>
            <el-descriptions-item label="插件依赖">{{
              formatBoolean(diagnoseCheck.pluginDependencyOk)
            }}</el-descriptions-item>
            <el-descriptions-item label="菜单冲突">{{
              formatBoolean(diagnoseCheck.menuConflictOk)
            }}</el-descriptions-item>
            <el-descriptions-item label="配置项">{{ diagnoseConfigTotal }}</el-descriptions-item>
          </el-descriptions>
          <el-table
            :data="validationItems"
            size="small"
            border
            empty-text="暂无诊断问题">
            <el-table-column
              label="分类"
              prop="category"
              width="120"
              align="center"/>
            <el-table-column
              label="等级"
              width="90"
              align="center">
              <template slot-scope="scope">
                <el-tag :type="getValidationLevelTagType(scope.row.level)">{{
                  getValidationLevelLabel(scope.row.level)
                }}</el-tag>
              </template>
            </el-table-column>
            <el-table-column
              label="类型"
              prop="kind"
              width="170"
              :show-overflow-tooltip="true"/>
            <el-table-column
              label="对象"
              prop="value"
              min-width="180"
              :show-overflow-tooltip="true"/>
            <el-table-column
              label="说明"
              prop="message"
              min-width="260"
              :show-overflow-tooltip="true"/>
          </el-table>
        </el-tab-pane>

        <el-tab-pane v-if="diagnose" label="菜单计划">
          <el-input
            :value="formatJson(diagnose.menuPlan)"
            type="textarea"
            :rows="12"
            readonly/>
        </el-tab-pane>

        <el-tab-pane label="原始数据">
          <el-input
            :value="formatJson(result)"
            type="textarea"
            :rows="14"
            readonly/>
        </el-tab-pane>
      </el-tabs>
    </template>

    <div class="dialog-footer" slot="footer">
      <el-button @click="emit('update:visible', false)">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script setup name="PluginDiagnosticDialog">
/**
 * 组件名称：PluginDiagnosticDialog
 * 功能说明：插件诊断弹窗，展示健康状态、检查摘要与原始数据
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}  visible                    - 是否显示弹窗
 * @prop {String}   title                      - 弹窗标题
 * @prop {Object}   result                     - 诊断结果
 * @prop {Boolean}  loading                    - 加载状态
 * @prop {Function} formatJson                 - 格式化 JSON
 * @prop {Function} formatBoolean              - 格式化布尔值
 * @prop {Function} getValidationLevelLabel    - 获取校验等级标签
 * @prop {Function} getValidationLevelTagType  - 获取校验等级标签类型
 *
 * 变量说明：
 * @data {Object} health              - 健康状态数据
 * @data {Object} diagnose            - 诊断数据
 * @data {Object} diagnoseCheck       - 诊断检查项
 * @data {Number} diagnoseConfigTotal - 配置项总数
 * @data {Array}  validationItems     - 校验明细列表
 *
 * 事件说明：
 * @event formatDuration - 格式化耗时
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
  title: { type: String, default: '插件诊断' },
  result: { type: Object, default: () => ({}) },
  loading: { type: Boolean, default: false },
  formatJson: { type: Function, required: true },
  formatBoolean: { type: Function, required: true },
  getValidationLevelLabel: { type: Function, required: true },
  getValidationLevelTagType: { type: Function, required: true },
})

const emit = defineEmits(['update:visible'])

/**
 * 健康状态数据
 * @returns {Object|null} 健康状态
 */
const health = computed(() => props.result.health || null)

/**
 * 诊断数据
 * @returns {Object|null} 诊断结果
 */
const diagnose = computed(() => (props.result.info || props.result.check ? props.result : null))

/**
 * 诊断检查项
 * @returns {Object} 检查项
 */
const diagnoseCheck = computed(() =>
  diagnose.value && diagnose.value.check ? diagnose.value.check : {}
)

/**
 * 配置项总数
 * @returns {Number|String} 配置项总数或占位符
 */
const diagnoseConfigTotal = computed(() => {
  const config = diagnose.value && diagnose.value.config
  const total = config && config.summary ? config.summary.total : undefined
  return total === null || total === undefined ? '-' : total
})

/**
 * 校验明细列表
 * @returns {Array} 校验明细
 */
const validationItems = computed(() => {
  const check = diagnose.value?.check || {}
  const groups = [
    ['依赖', check.dependencies],
    ['Manifest 错误', check.manifestIssues],
    ['Manifest 警告', check.manifestWarnings],
    ['插件依赖', check.pluginDependencyErrors],
    ['结构', check.structureErrors],
    ['菜单冲突', check.menuConflicts],
  ]

  return groups.flatMap(([category, items]) =>
    Array.isArray(items)
      ? items.map((item) => ({
          category,
          level: item.level || (category.includes('警告') ? 'warning' : 'error'),
          kind: item.kind || item.name || item.status || '-',
          value: item.value || item.requirement || item.path || item.pluginId || item.perms || '-',
          message: item.message || item.suggestion || item.reason || '',
        }))
      : []
  )
})

/**
 * 格式化耗时
 * @param {Number} value - 耗时（毫秒）
 * @returns {Number|String} 耗时或占位符
 */
function formatDuration(value)
{
  return value === null || value === undefined ? '-' : value
}
</script>