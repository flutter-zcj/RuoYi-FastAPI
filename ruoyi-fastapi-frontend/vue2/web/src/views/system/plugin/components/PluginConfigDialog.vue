<style lang="scss" scoped>
.config-help { width: 100%; margin-top: 4px; font-size: 12px; line-height: 18px; color: #909399; }
.config-help-text { display: block; }
.config-inline-actions { width: 100%; margin-top: 4px; line-height: 20px; }
.config-meta-list { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 6px; }
.plugin-config-control { width: 320px; max-width: 100%; }
.plugin-config-control ::v-deep .el-input { width: 100%; }
.plugin-config-item { margin-bottom: 18px; }
.plugin-config-section + .plugin-config-section { margin-top: 6px; }
.plugin-config-section-title { padding-left: 120px; margin: 0 0 10px; font-size: 13px; font-weight: 600; color: #606266; }
</style>

<template>
  <el-dialog
    :title="title"
    :visible="visible"
    width="720px"
    append-to-body
    @close="emit('update:visible', false)">
    <el-form
      ref="configRef"
      :model="configForm"
      label-width="120px">
      <el-empty v-if="!items.length" description="暂无插件配置"/>
      <div
        class="plugin-config-section"
        v-for="group in configGroups"
        :key="group.name">
        <div
          class="plugin-config-section-title"
          v-if="configGroups.length > 1 || group.name !== 'default'">
          {{ formatConfigGroupLabel(group.name) }}
        </div>
        <el-form-item
          class="plugin-config-item"
          v-for="item in group.items"
          :key="item.key"
          :label="item.label || item.key"
          :prop="'values.' + item.key"
          :rules="getConfigItemRules(item)">
          <el-switch
            v-if="item.type === 'boolean'"
            v-model="configForm.values[item.key]"/>
          <el-input-number
            class="plugin-config-control"
            v-else-if="item.type === 'number'"
            v-model="configForm.values[item.key]"
            :min="normalizeConfigBound(item.min)"
            :max="normalizeConfigBound(item.max)"
            :placeholder="getConfigInputPlaceholder(item)"/>
          <el-select
            class="plugin-config-control"
            v-else-if="item.type === 'select'"
            v-model="configForm.values[item.key]"
            :placeholder="getConfigInputPlaceholder(item)">
            <el-option
              v-for="option in item.options || []"
              :key="String(option.value)"
              :label="option.label"
              :value="option.value"/>
          </el-select>
          <template v-else-if="item.type === 'textarea' || item.type === 'json'">
            <el-input
              class="plugin-config-control"
              v-model="configForm.values[item.key]"
              type="textarea"
              :rows="4"
              :placeholder="getConfigInputPlaceholder(item)"/>
            <div class="config-inline-actions" v-if="item.type === 'json'">
              <el-button
                type="text"
                icon="el-icon-magic-stick"
                @click="formatConfigJsonValue(item)">格式化 JSON</el-button>
            </div>
          </template>
          <el-input
            class="plugin-config-control"
            v-else
            v-model="configForm.values[item.key]"
            :type="isSecretConfigItem(item) ? 'password' : 'text'"
            :show-password="isSecretConfigItem(item)"
            :placeholder="getConfigInputPlaceholder(item)"/>
          <div class="config-help">
            <span class="config-help-text" v-if="item.description">{{ item.description }}</span>
            <div class="config-meta-list" v-if="hasConfigMeta(item)">
              <el-tag
                size="small"
                type="info"
                effect="plain">{{ item.key }}</el-tag>
              <el-tag
                v-if="item.required"
                size="small"
                type="warning"
                effect="plain">必填</el-tag>
              <el-tag
                v-if="item.secret"
                size="small"
                type="warning"
                effect="plain">敏感</el-tag>
              <el-tag
                v-if="formatConfigDefaultValue(item) !== '-'"
                size="small"
                type="info"
                effect="plain">默认 {{ formatConfigDefaultValue(item) }}</el-tag>
              <el-tag
                v-if="formatConfigConstraint(item) !== '-'"
                size="small"
                type="info"
                effect="plain">{{ formatConfigConstraint(item) }}</el-tag>
            </div>
          </div>
        </el-form-item>
      </div>
    </el-form>
    <div class="dialog-footer" slot="footer">
      <el-button
        type="primary"
        :disabled="!items.length"
        :loading="loading"
        v-hasPermi="['system:plugin:edit']"
        @click="submit">保 存</el-button>
      <el-button @click="emit('update:visible', false)">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script setup name="PluginConfigDialog">
/**
 * 组件名称：PluginConfigDialog
 * 功能说明：插件配置弹窗，按分组渲染配置项并提交配置值
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean}  visible                    - 是否显示弹窗
 * @prop {String}   title                      - 弹窗标题
 * @prop {Array}    items                      - 配置项列表
 * @prop {Boolean}  loading                    - 加载状态
 * @prop {Function} formatConfigDefaultValue   - 格式化默认值
 * @prop {Function} formatConfigConstraint     - 格式化约束
 *
 * 变量说明：
 * @data {Object} configRef    - 表单引用
 * @data {Object} configForm   - 配置表单数据
 * @data {Array}  configGroups - 按分组聚合后的配置项
 *
 * 事件说明：
 * @event submit                     - 提交配置值
 * @event normalizeConfigValueForForm - 规范化配置项表单值
 * @event buildConfigSubmitValues    - 构建提交数据
 * @event getConfigItemRules         - 获取配置项校验规则
 * @event isSecretConfigItem         - 判断是否为敏感配置项
 * @event buildNumberConfigValidator - 构建数字校验器
 * @event buildPatternConfigValidator - 构建正则校验器
 * @event validateJsonConfig         - 校验 JSON 配置
 * @event formatConfigJsonValue      - 格式化 JSON 配置值
 * @event getConfigInputPlaceholder  - 获取配置输入占位符
 * @event normalizeConfigBound       - 规范化配置边界值
 * @event hasConfigMeta              - 是否存在配置元信息
 * @event formatConfigGroupLabel     - 格式化配置分组标签
 *
 * 插槽说明：
 */

/**
 * 2026/07/28     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { computed, reactive, ref, watch } from 'vue'
import { Message } from 'element-ui'

const props = defineProps({
  visible: { type: Boolean, default: false },
  title: { type: String, default: '插件配置' },
  items: { type: Array, default: () => [] },
  loading: { type: Boolean, default: false },
  formatConfigDefaultValue: { type: Function, required: true },
  formatConfigConstraint: { type: Function, required: true },
})

const emit = defineEmits(['update:visible', 'submit'])
const configRef = ref(null)
const configForm = reactive({
  values: {},
})

/**
 * 按分组聚合后的配置项
 * @returns {Array} 分组配置项
 */
const configGroups = computed(() => {
  const groups = new Map()
  const sortedItems = [...props.items].sort((left, right) => {
    const leftGroup = left.group || 'default'
    const rightGroup = right.group || 'default'
    if (leftGroup !== rightGroup) {
      return leftGroup.localeCompare(rightGroup)
    }
    const leftOrder = Number(left.order || 0)
    const rightOrder = Number(right.order || 0)
    if (leftOrder !== rightOrder) {
      return leftOrder - rightOrder
    }
    return String(left.key || '').localeCompare(String(right.key || ''))
  })
  sortedItems.forEach((item) => {
    const groupName = item.group || 'default'
    if (!groups.has(groupName)) {
      groups.set(groupName, [])
    }
    groups.get(groupName).push(item)
  })
  return Array.from(groups, ([name, items]) => ({ name, items }))
})

/**
 * 监听配置项变化，重新初始化表单值
 */
watch(
  () => props.items,
  (items) => {
    const values = {}
    items.forEach((item) => {
      values[item.key] = normalizeConfigValueForForm(item)
    })
    configForm.values = values
  },
  { immediate: true }
)

/**
 * 提交配置值
 */
function submit()
{
  configRef.value?.validate((valid) => {
    if (!valid) {
      return
    }
    let values
    try {
      values = buildConfigSubmitValues()
    } catch (error) {
      Message.error(error.message || '配置格式不正确')
      return
    }
    emit('submit', values)
  })
}

/**
 * 规范化配置项表单值
 * @param {Object} item - 配置项
 * @returns {*} 表单值
 */
function normalizeConfigValueForForm(item)
{
  if (item.type !== 'json') {
    return item.value
  }
  if (item.value === null || item.value === undefined || item.value === '') {
    return ''
  }
  if (typeof item.value === 'string') {
    return item.value
  }
  return JSON.stringify(item.value, null, 2)
}

/**
 * 构建提交数据
 * @returns {Object} 提交配置值
 */
function buildConfigSubmitValues()
{
  return props.items.reduce((values, item) => {
    const value = configForm.values[item.key]
    if (item.type === 'json' && typeof value === 'string' && value.trim()) {
      try {
        values[item.key] = JSON.parse(value)
      } catch {
        throw new Error((item.label || item.key) + ' 不是合法 JSON')
      }
      return values
    }
    values[item.key] = value
    return values
  }, {})
}

/**
 * 获取配置项校验规则
 * @param {Object} item - 配置项
 * @returns {Array} 校验规则
 */
function getConfigItemRules(item)
{
  const rules = []
  if (item.required) {
    rules.push({
      required: true,
      message: '不能为空',
      trigger: item.type === 'select' ? 'change' : 'blur',
    })
  }
  if (item.type === 'number') {
    rules.push({ validator: buildNumberConfigValidator(item), trigger: 'change' })
  }
  if (item.pattern && ['string', 'textarea', 'password'].includes(item.type)) {
    rules.push({ validator: buildPatternConfigValidator(item), trigger: 'blur' })
  }
  if (item.type === 'json') {
    rules.push({ validator: validateJsonConfig, trigger: 'blur' })
  }
  return rules
}

/**
 * 判断是否为敏感配置项
 * @param {Object} item - 配置项
 * @returns {Boolean} 是否为敏感配置项
 */
function isSecretConfigItem(item)
{
  return item.type === 'password' || item.secret
}

/**
 * 构建数字校验器
 * @param {Object} item - 配置项
 * @returns {Function} 校验器
 */
function buildNumberConfigValidator(item)
{
  return (_rule, value, callback) => {
    if (value === undefined || value === null || value === '') {
      callback()
      return
    }
    const numberValue = Number(value)
    if (Number.isNaN(numberValue)) {
      callback(new Error('请输入数字'))
      return
    }
    if (item.min !== undefined && item.min !== null && numberValue < Number(item.min)) {
      callback(new Error('不能小于 ' + item.min))
      return
    }
    if (item.max !== undefined && item.max !== null && numberValue > Number(item.max)) {
      callback(new Error('不能大于 ' + item.max))
      return
    }
    callback()
  }
}

/**
 * 构建正则校验器
 * @param {Object} item - 配置项
 * @returns {Function} 校验器
 */
function buildPatternConfigValidator(item)
{
  return (_rule, value, callback) => {
    if (value === undefined || value === null || value === '') {
      callback()
      return
    }
    try {
      if (!new RegExp(item.pattern).test(String(value))) {
        callback(new Error('格式不符合要求'))
        return
      }
    } catch {
      callback(new Error('配置正则表达式无效'))
      return
    }
    callback()
  }
}

/**
 * 校验 JSON 配置
 * @param {Object} _rule - 校验规则
 * @param {String} value - 配置值
 * @param {Function} callback - 回调函数
 */
function validateJsonConfig(_rule, value, callback)
{
  if (value === undefined || value === null || value === '') {
    callback()
    return
  }
  try {
    JSON.parse(value)
    callback()
  } catch {
    callback(new Error('请输入合法 JSON'))
  }
}

/**
 * 格式化 JSON 配置值
 * @param {Object} item - 配置项
 */
function formatConfigJsonValue(item)
{
  const value = configForm.values[item.key]
  if (value === undefined || value === null || value === '') {
    return
  }
  try {
    configForm.values[item.key] = JSON.stringify(
      typeof value === 'string' ? JSON.parse(value) : value,
      null,
      2
    )
  } catch {
    Message.error((item.label || item.key) + ' 不是合法 JSON')
  }
}

/**
 * 获取配置输入占位符
 * @param {Object} item - 配置项
 * @returns {String} 占位符
 */
function getConfigInputPlaceholder(item)
{
  return item.placeholder || item.description || ''
}

/**
 * 规范化配置边界值
 * @param {*} value - 边界值
 * @returns {*} 规范化后的边界值
 */
function normalizeConfigBound(value)
{
  return value === null || value === undefined ? undefined : value
}

/**
 * 是否存在配置元信息
 * @param {Object} item - 配置项
 * @returns {Boolean} 是否存在元信息
 */
function hasConfigMeta(item)
{
  return Boolean(
    item.key ||
      item.required ||
      item.secret ||
      props.formatConfigDefaultValue(item) !== '-' ||
      props.formatConfigConstraint(item) !== '-'
  )
}

/**
 * 格式化配置分组标签
 * @param {String} groupName - 分组名称
 * @returns {String} 分组标签
 */
function formatConfigGroupLabel(groupName)
{
  const groupMap = {
    default: '基础配置',
    model: '模型配置',
    security: '安全配置',
    advanced: '高级配置',
  }
  return groupMap[groupName] || groupName
}
</script>