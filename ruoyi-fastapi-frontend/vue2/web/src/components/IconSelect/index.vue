<style lang="scss" scoped>
.icon-body { width: 100%; padding: 10px;
  .icon-search { position: relative; margin-bottom: 5px; }
  .icon-list { height: 200px; overflow: auto;
    .list-container { display: flex; flex-wrap: wrap;
      .icon-item-wrapper { display: flex; width: calc(100% / 3); height: 25px; line-height: 25px; cursor: pointer;
        .icon-item { display: flex; max-width: 100%; height: 100%; padding: 0 5px;
          &:hover { background: #ececec; border-radius: 5px; }
          .icon { flex-shrink: 0; }
          span { display: inline-block; overflow: hidden; padding-left: 2px; vertical-align: -0.15em; text-overflow: ellipsis; white-space: nowrap; fill: currentColor; }
        }
        .icon-item.active { background: #ececec; border-radius: 5px; }
      }
    }
  }
}
</style>

<!-- @author zhengjie -->
<template>
  <div class="icon-body">
    <el-input
      class="icon-search"
      v-model="name"
      clearable
      placeholder="请输入图标名称"
      @clear="filterIcons"
      @input="filterIcons">
      <i class="el-icon-search el-input__icon" slot="suffix" />
    </el-input>
    <div class="icon-list">
      <div class="list-container">
        <div
          class="icon-item-wrapper"
          v-for="(item, index) in iconList"
          :key="index"
          @click="selectedIcon(item)">
          <div :class="['icon-item', { active: activeIcon === item }]">
            <svg-icon
              :icon-class="item"
              class-name="icon"
              style="height: 25px; width: 16px"/>
            <span>{{ item }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：IconSelect
 * 功能说明：图标选择组件，支持图标搜索与选中
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} activeIcon - 当前选中的图标
 *
 * 变量说明：
 *
 * 事件说明：
 * @event selected - 选中图标
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import icons from './requireIcons'

export default
{
  name: 'IconSelect',

  props:
  {
    activeIcon: { type: String }
  },

  data()
  {
    return {
      name: '',
      iconList: icons,
    }
  },

  methods:
  {
    /**
     * 根据名称过滤图标列表
     */
    filterIcons()
    {
      this.iconList = icons
      if (this.name)
      {
        this.iconList = this.iconList.filter((item) => item.includes(this.name))
      }
    },

    /**
     * 选中图标并向上抛出选中事件
     * @param {String} name - 图标名称
     */
    selectedIcon(name)
    {
      this.$emit('selected', name)
      document.body.click()
    },

    /**
     * 重置搜索名称与图标列表
     */
    reset()
    {
      this.name = ''
      this.iconList = icons
    }
  }
}
</script>