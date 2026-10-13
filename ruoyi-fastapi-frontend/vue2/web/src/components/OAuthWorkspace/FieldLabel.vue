<style lang="scss" scoped>
.field-label { display: inline-flex; align-items: center; gap: 5px; white-space: nowrap; }
.field-help { flex: 0 0 auto; color: var(--oauth-text-color-secondary); cursor: help; outline: none; }
.field-help:hover, .field-help:focus-visible { color: var(--oauth-color-primary); }
.field-help:focus-visible { border-radius: 50%; box-shadow: 0 0 0 2px var(--oauth-color-primary-light-7); }
</style>

<template>
  <span class="field-label">
    <span>{{ label }}</span>
    <el-tooltip
      :content="help"
      placement="top"
      :open-delay="250">
      <i
        class="field-help el-icon-question"
        tabindex="0"
        :aria-label="`${label}说明`"
        @click.stop></i>
    </el-tooltip>
  </span>
</template>

<script setup>
/**
 * 组件名称：OAuthWorkspace
 * 功能说明：字段标签组件，展示字段名称并提供帮助提示
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} label - 字段名称
 * @prop {String} help  - 字段帮助提示内容
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

defineProps({
  label: { type: String, required: true },
  help: { type: String, required: true },
})
</script>