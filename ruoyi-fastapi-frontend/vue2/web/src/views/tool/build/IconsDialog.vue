<style lang="scss" scoped>
.icon-ul { padding: 0; margin: 0; font-size: 0;
  li { display: inline-block; overflow: hidden; width: 16.66%; height: 108px; padding: 15px 6px 6px 6px; font-size: 14px; text-align: center; list-style-type: none; box-sizing: border-box; cursor: pointer;
    &:hover { background: #f2f2f2; }
    &.active-item { background: #e1f3fb; color: #7a6df0; }
    > i { font-size: 30px; line-height: 50px; }
  }
}
.icon-dialog {
  ::v-deep .el-dialog { display: flex; flex-direction: column; overflow: hidden; max-height: 92vh; margin-bottom: 0; margin-top: 4vh !important; border-radius: 8px; box-sizing: border-box;
    .el-dialog__header { padding-top: 14px; }
    .el-dialog__body { overflow: auto; padding: 0; margin: 0 20px 20px 20px; }
  }
}
</style>

<template>
  <div class="icon-dialog">
    <el-dialog
      width="980px"
      :modal-append-to-body="false"
      v-bind="$attrs"
      v-on="$listeners"
      @open="onOpen"
      @close="onClose">
      <div slot="title">
        选择图标
        <el-input
          v-model="key"
          size="mini"
          :style="{ width: '260px' }"
          placeholder="请输入图标名称"
          prefix-icon="el-icon-search"
          clearable/>
      </div>
      <ul class="icon-ul">
        <li
          v-for="icon in iconList"
          :key="icon"
          :class="active === icon ? 'active-item' : ''"
          @click="onSelect(icon)">
          <i :class="icon" />
          <div>{{ icon }}</div>
        </li>
      </ul>
    </el-dialog>
  </div>
</template>

<script>
/**
 * 组件名称：IconsDialog
 * 功能说明：图标选择弹窗组件，支持按名称搜索并选择 Element 图标
 * 研发时间：2024/12/11
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} current - 当前选中的图标名称
 *
 * 变量说明：
 * @data {Array}  iconList - 图标列表
 * @data {String} active   - 当前选中的图标名称
 * @data {String} key      - 搜索关键词
 *
 * 事件说明：
 * @event onOpen   - 弹窗打开时初始化选中项与搜索词
 * @event onClose  - 弹窗关闭回调
 * @event onSelect - 选中图标并抛出事件
 *
 * 插槽说明：
 */

/**
 * 2024/12/11     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import iconList from '@/utils/generator/icon.json'

const originList = iconList.map((name) => `el-icon-${name}`)

export default
{
  inheritAttrs: false,

  props: ['current'],

  data()
  {
    return {
      iconList: originList,
      active: null,
      key: '',
    }
  },

  watch:
  {
    /**
     * 监听搜索关键词变化，过滤图标列表
     * @param {String} val - 搜索关键词
     */
    key(val)
    {
      if (val)
      {
        this.iconList = originList.filter((name) => name.indexOf(val) > -1)
      }
      else
      {
        this.iconList = originList
      }
    },
  },

  methods:
  {
    /**
     * 弹窗打开时初始化选中项与搜索词
     */
    onOpen()
    {
      this.active = this.current
      this.key = ''
    },

    /**
     * 弹窗关闭回调
     */
    onClose() {},

    /**
     * 选中图标并抛出事件
     * @param {String} icon - 图标名称
     */
    onSelect(icon)
    {
      this.active = icon
      this.$emit('select', icon)
      this.$emit('update:visible', false)
    },
  },
}
</script>