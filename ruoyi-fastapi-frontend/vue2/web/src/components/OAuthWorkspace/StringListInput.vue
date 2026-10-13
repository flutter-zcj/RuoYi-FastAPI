<style lang="scss" scoped>
.string-list-input { width: 100%; }
.string-list-row { display: grid; margin-bottom: 8px; grid-template-columns: minmax(0, 1fr) 36px; gap: 6px; }
.string-list-row :deep(.el-button) { width: 36px; margin: 0; }
.add-row { margin-left: 0; }
</style>

<template>
  <div class="string-list-input">
    <div
      class="string-list-row"
      v-for="(_item, index) in localItems"
      :key="index">
      <el-input
        :placeholder="placeholder"
        v-model="localItems[index]"
        @blur="emitValue"
        @keyup.enter.native="appendAfter(index)"/>
      <el-button
        class="oauth-button-danger"
        type="text"
        icon="el-icon-delete"
        :aria-label="`删除第 ${index + 1} 项`"
        @click="remove(index)"/>
    </div>
    <el-button
      class="add-row"
      type="text"
      icon="el-icon-plus"
      @click="append">
      {{ addText }}
    </el-button>
  </div>
</template>

<script setup>
/**
 * 组件名称：StringListInput
 * 功能说明：字符串列表输入组件，支持按行输入、回车插入与删除并同步绑定值
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Array}  value       - 绑定值
 * @prop {String} placeholder - 输入框占位符
 * @prop {String} addText     - 新增按钮文本
 *
 * 变量说明：
 *
 * 事件说明：
 * @event input - 列表值变化
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { ref, watch } from 'vue'
const props = defineProps({
  value: { type: Array, default: () => [] },
  placeholder: { type: String, default: '' },
  addText: { type: String, default: '添加一项' },
})
const emit = defineEmits(['input'])
const localItems = ref([])

/**
 * 同步外部列表，空列表保留一个输入行
 * @param {Array} value - 外部列表值
 */
function syncFromValue(value)
{
  const items = Array.isArray(value) ? value.map((item) => String(item ?? '')) : []
  localItems.value = items.length ? items : ['']
}

/**
 * 过滤空值并更新列表绑定
 */
function emitValue()
{
  emit('input', localItems.value.map((item) => item.trim()).filter(Boolean))
}

/**
 * 新增输入行
 */
function append()
{
  localItems.value.push('')
}

/**
 * 在当前非空输入行后插入一行
 * @param {Number} index - 当前行下标
 */
function appendAfter(index)
{
  if (!localItems.value[index]?.trim())
  {
    return
  }
  localItems.value.splice(index + 1, 0, '')
}

/**
 * 删除输入行并更新列表绑定
 * @param {Number} index - 待删除行下标
 */
function remove(index)
{
  localItems.value.splice(index, 1)
  if (!localItems.value.length)
  {
    localItems.value.push('')
  }
  emitValue()
}

watch(() => props.value, syncFromValue, { immediate: true, deep: true })
</script>