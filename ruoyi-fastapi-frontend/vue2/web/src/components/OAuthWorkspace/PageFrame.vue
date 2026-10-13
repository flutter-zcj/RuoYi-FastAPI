<style lang="scss" scoped>
.oauth-workspace { --workspace-surface: var(--oauth-bg-color-overlay); --workspace-border: var(--oauth-border-color-lighter); min-height: calc(100vh - 84px); background: transparent; }
.workspace-header { display: flex; padding: 0 0 12px; align-items: center; justify-content: space-between; gap: 20px; }
.workspace-heading { display: flex; flex: 1; align-items: center; gap: 12px; min-width: 0; }
.workspace-section { display: inline-flex; height: 18px; padding-left: 12px; border-left: 1px solid var(--oauth-border-color); color: var(--oauth-color-primary); font-size: 12px; font-weight: 600; letter-spacing: 0; flex: 0 0 auto; align-items: center; }
.workspace-heading h2 { margin: 0; color: var(--oauth-text-color-primary); font-size: 20px; font-weight: 650; flex: 0 0 auto; line-height: 30px; }
.workspace-heading p { overflow: hidden; margin: 0; color: var(--oauth-text-color-secondary); font-size: 13px; text-overflow: ellipsis; white-space: nowrap; flex: 1; min-width: 0; line-height: 1.4; }
.workspace-actions { display: flex; padding-top: 0; flex: 0 0 auto; align-items: center; gap: 10px; }
.workspace-context, .workspace-filters, .workspace-data { margin-bottom: 16px; background: var(--workspace-surface); border: 1px solid var(--workspace-border); border-radius: 10px; }
.workspace-context { padding: 18px 20px; }
.workspace-filters { padding: 16px 18px 4px; }
.filter-heading { display: flex; margin-bottom: 14px; align-items: center; justify-content: space-between; min-height: 24px; }
.filter-title { color: var(--oauth-text-color-primary); font-size: 14px; font-weight: 600; }
.filter-hint { margin-left: 10px; color: var(--oauth-text-color-secondary); font-size: 12px; }
.workspace-filters :deep(.el-form-item) { margin-bottom: 14px; }
.workspace-data { overflow: hidden; }
.data-toolbar { display: flex; padding: 0 16px; border-bottom: 1px solid var(--workspace-border); align-items: center; justify-content: space-between; min-height: 54px; }
.workspace-data :deep(.el-table) { --oauth-table-header-bg-color: var(--oauth-fill-color-light); }
.workspace-data :deep(.el-table th.el-table__cell) { height: 46px; color: var(--oauth-text-color-regular); font-weight: 600; }
.workspace-data :deep(.el-table td.el-table__cell) { height: 58px; }
.workspace-data :deep(.table-actions) { display: flex; white-space: nowrap; align-items: center; justify-content: center; gap: 12px; }
.workspace-data :deep(.table-actions .el-button + .el-button) { margin-left: 0; }
.workspace-data :deep(.table-actions .el-dropdown) { flex: 0 0 auto; }
.workspace-data :deep(.pagination-container) { margin: 0; padding: 15px 16px; background: transparent; border-top: 1px solid var(--workspace-border); }
@media (max-width: 768px) {
  .oauth-workspace { min-height: 100%; }
  .workspace-header { display: block; }
  .workspace-heading { flex-wrap: wrap; gap: 4px 10px; }
  .workspace-heading p { overflow: visible; white-space: normal; flex-basis: 100%; }
  .workspace-actions { padding-top: 10px; }
  .workspace-filters { overflow-x: auto; }
}
@media (prefers-reduced-motion: reduce) {
  .oauth-workspace *, .oauth-workspace *::before, .oauth-workspace *::after { scroll-behavior: auto !important; transition-duration: 0.01ms !important; }
}
</style>

<template>
  <div class="app-container oauth-workspace">
    <header class="workspace-header">
      <div class="workspace-heading">
        <h2>{{ title }}</h2>
        <span class="workspace-section">{{ section }}</span>
        <p>{{ description }}</p>
      </div>
      <div class="workspace-actions" v-if="$slots.actions || $scopedSlots.actions">
        <slot name="actions" />
      </div>
    </header>

    <section
      class="workspace-context"
      aria-label="页面概览"
      v-if="$slots.context || $scopedSlots.context">
      <slot name="context" />
    </section>

    <section
      class="workspace-filters"
      aria-label="筛选条件"
      v-if="$slots.filters || $scopedSlots.filters">
      <div class="filter-heading">
        <div>
          <span class="filter-title">筛选条件</span>
          <span class="filter-hint" v-if="filterHint">{{ filterHint }}</span>
        </div>
        <slot name="filter-extra" />
      </div>
      <slot name="filters" />
    </section>

    <section class="workspace-data">
      <div class="data-toolbar" v-if="$slots.toolbar || $scopedSlots.toolbar">
        <slot name="toolbar" />
      </div>
      <slot />
    </section>
  </div>
</template>

<script setup>
/**
 * 组件名称：PageFrame
 * 功能说明：统一认证页面框架组件，提供页头、概览、筛选与数据区布局及插槽
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} section     - 所属模块名称
 * @prop {String} title       - 页面标题
 * @prop {String} description - 页面描述
 * @prop {String} filterHint  - 筛选条件提示
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
  section: { type: String, default: '统一认证' },
  title: { type: String, required: true },
  description: { type: String, required: true },
  filterHint: { type: String, default: '' },
})
</script>