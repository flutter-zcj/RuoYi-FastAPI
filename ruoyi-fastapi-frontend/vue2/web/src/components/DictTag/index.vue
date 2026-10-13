<style lang="scss" scoped>
.el-tag + .el-tag { margin-left: 10px; }
</style>

<template>
  <div>
    <template v-for="(item, index) in options">
      <template v-if="isValueMatch(item.value)">
        <span
          v-if="
            (item.raw.listClass == 'default' || item.raw.listClass == '') &&
            (item.raw.cssClass == '' || item.raw.cssClass == null)
          "
          :key="item.value"
          :index="index"
          :class="item.raw.cssClass">{{ item.label + ' ' }}</span>
        <el-tag
          v-else
          :disable-transitions="true"
          :key="item.value"
          :index="index"
          :type="item.raw.listClass == 'primary' ? '' : item.raw.listClass"
          :class="item.raw.cssClass">
          {{ item.label + ' ' }}
        </el-tag>
      </template>
    </template>
    <template v-if="unmatch && showValue">
      {{ unmatchArray | handleArray }}
    </template>
  </div>
</template>

<script>
/**
 * 组件名称：DictTag
 * 功能说明：字典标签组件，根据字典选项渲染标签并展示未匹配值
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Array}                   options   - 字典选项
 * @prop {[Number, String, Array]} value     - 当前值
 * @prop {Boolean}                 showValue - 是否显示未匹配值
 * @prop {String}                  separator - 值分隔符
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'DictTag',

  props:
  {
    options: { type: Array, default: null },
    value: [Number, String, Array],
    showValue: { type: Boolean, default: true },
    separator: { type: String, default: ',' }
  },

  data()
  {
    return {
      unmatchArray: [], // 记录未匹配的项
    }
  },

  computed:
  {
    /**
     * 将传入值标准化为数组
     * @returns {Array} 值数组
     */
    values()
    {
      if (this.value === null || typeof this.value === 'undefined' || this.value === '') return []
      if (typeof this.value === 'number' || typeof this.value === 'boolean') return [this.value]
      return Array.isArray(this.value)
        ? this.value.map((item) => '' + item)
        : String(this.value).split(this.separator)
    },

    /**
     * 判断并收集未匹配的字典项
     * @returns {Boolean} 是否存在未匹配项
     */
    unmatch()
    {
      this.unmatchArray = []
      // 没有value不显示
      if (
        this.value === null ||
        typeof this.value === 'undefined' ||
        this.value === '' ||
        this.options.length === 0
      )
        return false
      // 传入值为数组
      let unmatch = false // 添加一个标志来判断是否有未匹配项
      this.values.forEach((item) =>
      {
        if (!this.options.some((v) => v.value == item))
        {
          this.unmatchArray.push(item)
          unmatch = true // 如果有未匹配项，将标志设置为true
        }
      })
      return unmatch // 返回标志的值
    }
  },

  methods:
  {
    /**
     * 判断字典项是否与当前值匹配
     * @param {String} itemValue - 字典项的值
     * @returns {Boolean} 是否匹配
     */
    isValueMatch(itemValue)
    {
      return this.values.some((val) => val == itemValue)
    }
  },

  filters:
  {
    /**
     * 将数组拼接为空格分隔的字符串
     * @param {Array} array - 待处理数组
     * @returns {String} 拼接结果
     */
    handleArray(array)
    {
      if (array.length === 0) return ''
      return array.reduce((pre, cur) =>
      {
        return pre + ' ' + cur
      })
    }
  }
}
</script>