<style lang="scss" scoped>
.mb16 { margin-bottom: 16px; }
.mr12 { margin-right: 12px; }
.plan-summary { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 10px; margin-bottom: 16px; }
.plan-summary-item { min-width: 0; padding: 10px 12px; border: 1px solid #ebeef5; background: #f8f9fb; border-radius: 4px; }
.plan-summary-label { display: block; margin-bottom: 6px; font-size: 12px; color: #909399; }
.plan-summary-value { font-size: 14px; font-weight: 600; color: #303133; word-break: break-all; }
.plan-summary-value.is-danger { color: #f56c6c; }
.plan-order-tag { margin-right: 6px; margin-bottom: 4px; }
@media (max-width: 1200px) {
  .plan-summary { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
@media (max-width: 768px) {
  .plan-summary { grid-template-columns: 1fr; }
}
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
      v-if="planResult.message"
      :title="planResult.message"
      :type="planResult.ok ? 'success' : 'warning'"
      show-icon
      :closable="false"/>
    <div class="plan-summary">
      <div class="plan-summary-item">
        <span class="plan-summary-label">计划操作</span>
        <span class="plan-summary-value">{{ formatPluginOperation(planResult.operation) }}</span>
      </div>
      <div class="plan-summary-item">
        <span class="plan-summary-label">目标插件</span>
        <span class="plan-summary-value">{{ planResult.requestedPluginIds.length }}</span>
      </div>
      <div class="plan-summary-item">
        <span class="plan-summary-label">可执行</span>
        <span class="plan-summary-value">{{ planResult.executablePluginIds.length }}</span>
      </div>
      <div class="plan-summary-item">
        <span class="plan-summary-label">阻塞项</span>
        <span class="plan-summary-value is-danger">{{ planResult.blockerCount }}</span>
      </div>
    </div>
    <el-descriptions
      class="mb16"
      :column="1"
      border>
      <el-descriptions-item label="目标范围">{{
        formatPluginIdsForDisplay(planResult.requestedPluginIds)
      }}</el-descriptions-item>
      <el-descriptions-item label="执行顺序">
        <el-tag
          class="plan-order-tag"
          v-for="pluginId in planResult.executablePluginIds"
          :key="pluginId"
          type="info">{{ pluginId }}</el-tag>
        <span v-if="!planResult.executablePluginIds.length">-</span>
      </el-descriptions-item>
    </el-descriptions>

    <el-tabs>
      <el-tab-pane label="计划项">
        <el-table
          :data="planResult.items"
          size="small"
          border
          empty-text="暂无计划项">
          <el-table-column
            label="顺序"
            prop="order"
            width="70"
            align="center"/>
          <el-table-column
            label="插件ID"
            prop="pluginId"
            width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="插件名称"
            prop="name"
            min-width="150"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="版本"
            prop="version"
            width="100"
            align="center"/>
          <el-table-column
            label="显式选择"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.requested ? 'success' : 'info'">{{
                scope.row.requested ? '是' : '依赖'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="可执行"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="getPlanReadyTagType(scope.row.ready)">{{
                scope.row.ready ? '是' : '否'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="依赖"
            min-width="180"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">
              <span>{{ formatPlanDependencies(scope.row.dependencies) }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="阻塞项"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="getBlockerCount(scope.row) ? 'danger' : 'success'">{{
                getBlockerCount(scope.row)
              }}</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="阻塞原因">
        <el-table
          :data="planResult.blockers"
          size="small"
          border
          empty-text="暂无阻塞项">
          <el-table-column
            label="插件ID"
            prop="pluginId"
            width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="依赖插件"
            prop="dependencyId"
            width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="状态"
            width="150"
            align="center">
            <template slot-scope="scope">
              <el-tag type="danger">{{ getPlanBlockerStatusLabel(scope.row.status) }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="message"
            min-width="320"
            :show-overflow-tooltip="true"/>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="执行结果">
        <el-descriptions
          class="mb16"
          :column="4"
          border>
          <el-descriptions-item label="总数">{{ batchResult.summary.total }}</el-descriptions-item>
          <el-descriptions-item label="成功">{{
            batchResult.summary.succeeded
          }}</el-descriptions-item>
          <el-descriptions-item label="失败">{{ batchResult.summary.failed }}</el-descriptions-item>
          <el-descriptions-item label="跳过">{{
            batchResult.summary.skipped
          }}</el-descriptions-item>
        </el-descriptions>
        <el-table
          :data="batchResult.executed"
          size="small"
          border
          empty-text="暂无执行记录">
          <el-table-column
            label="插件ID"
            prop="pluginId"
            width="140"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="操作"
            width="90"
            align="center">
            <template slot-scope="scope">{{ formatPluginOperation(scope.row.operation) }}</template>
          </el-table-column>
          <el-table-column
            label="状态"
            width="90"
            align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.ok ? 'success' : 'danger'">{{
                scope.row.status || '-'
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="耗时"
            prop="durationMs"
            width="90"
            align="center">
            <template slot-scope="scope">{{ formatDuration(scope.row.durationMs) }}ms</template>
          </el-table-column>
          <el-table-column
            label="说明"
            prop="message"
            min-width="220"
            :show-overflow-tooltip="true"/>
          <el-table-column
            label="建议"
            prop="suggestion"
            min-width="260"
            :show-overflow-tooltip="true"/>
        </el-table>
      </el-tab-pane>
    </el-tabs>

    <div class="dialog-footer" slot="footer">
      <el-switch
        class="mr12"
        :value="continueOnError"
        active-text="失败后继续"
        inactive-text="失败即中止"
        @input="emit('update:continueOnError', $event)"/>
      <el-button
        type="primary"
        :disabled="!canExecuteBatchPlan"
        :loading="loading"
        v-hasPermi="['system:plugin:edit']"
        @click="emit('execute')">执行计划：{{ formatPluginOperation(planResult.operation) }}</el-button>
      <el-button @click="emit('update:visible', false)">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script setup name="PluginPlanDialog">
/**
 * 组件名称：PluginPlanDialog
 * 功能说明：插件依赖计划弹窗，展示计划项、阻塞原因与执行结果
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}  visible                     - 是否显示弹窗
 * @prop {String}   title                       - 弹窗标题
 * @prop {Object}   planResult                  - 依赖计划结果
 * @prop {Object}   batchResult                 - 批量执行结果
 * @prop {Boolean}  continueOnError             - 失败后是否继续
 * @prop {Boolean}  loading                     - 加载状态
 * @prop {Function} formatPluginOperation       - 格式化插件操作
 * @prop {Function} formatPluginIdsForDisplay   - 格式化插件ID列表展示
 * @prop {Function} formatPlanDependencies      - 格式化计划依赖
 * @prop {Function} getPlanReadyTagType         - 获取可执行标签类型
 * @prop {Function} getPlanBlockerStatusLabel   - 获取阻塞状态标签
 *
 * 变量说明：
 * @data {Boolean} canExecuteBatchPlan - 是否可执行批量计划
 *
 * 事件说明：
 * @event getBlockerCount - 获取阻塞项数量
 * @event formatDuration  - 格式化耗时
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
  title: { type: String, default: '插件依赖计划' },
  planResult: { type: Object, required: true },
  batchResult: { type: Object, required: true },
  continueOnError: { type: Boolean, default: false },
  loading: { type: Boolean, default: false },
  formatPluginOperation: { type: Function, required: true },
  formatPluginIdsForDisplay: { type: Function, required: true },
  formatPlanDependencies: { type: Function, required: true },
  getPlanReadyTagType: { type: Function, required: true },
  getPlanBlockerStatusLabel: { type: Function, required: true },
})

const emit = defineEmits(['update:visible', 'update:continueOnError', 'execute'])

/**
 * 是否可执行批量计划
 * @returns {Boolean} 是否可执行
 */
const canExecuteBatchPlan = computed(() => {
  return props.planResult.ok && props.planResult.executablePluginIds.length > 0
})

/**
 * 获取阻塞项数量
 * @param {Object} row - 计划项数据
 * @returns {Number} 阻塞项数量
 */
function getBlockerCount(row)
{
  return Array.isArray(row.blockers) ? row.blockers.length : 0
}

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