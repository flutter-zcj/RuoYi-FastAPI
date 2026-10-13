<style lang="scss" scoped></style>

<template>
  <el-form size="small">
    <el-form-item>
      <el-radio v-model="radioValue" :label="1">
        小时，允许的通配符[, - * /]
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="2">
        周期从
        <el-input-number
          v-model="cycle01"
          :min="0"
          :max="22"/>
        -
        <el-input-number
          v-model="cycle02"
          :min="cycle01 ? cycle01 + 1 : 1"
          :max="23"/>
        小时
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="3">
        从
        <el-input-number
          v-model="average01"
          :min="0"
          :max="22"/>
        小时开始，每
        <el-input-number
          v-model="average02"
          :min="1"
          :max="23 - average01 || 0"/>
        小时执行一次
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="4">
        指定
        <el-select
          clearable
          v-model="checkboxList"
          placeholder="可多选"
          multiple
          style="width: 100%">
          <el-option
            v-for="item in 24"
            :key="item"
            :value="item - 1">{{ item - 1 }}</el-option>
        </el-select>
      </el-radio>
    </el-form-item>
  </el-form>
</template>

<script>
/**
 * 组件名称：Crontab
 * 功能说明：Cron 表达式小时字段配置组件
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Function} check - 数字格式校验方法
 * @prop {Object}   cron  - Cron 表达式对象
 *
 * 变量说明：
 * @data {Number}   radioValue   - 当前选中的单选值
 * @data {Number}   cycle01      - 周期起始值
 * @data {Number}   cycle02      - 周期结束值
 * @data {Number}   average01    - 平均起始值
 * @data {Number}   average02    - 平均间隔值
 * @data {Array}    checkboxList - 指定勾选值列表
 * @data {Function} checkNum     - 数字校验方法
 *
 * 事件说明：
 * @event radioChange    - 单选按钮值变化时
 * @event cycleChange    - 周期两个值变化时
 * @event averageChange  - 平均两个值变化时
 * @event checkboxChange - checkbox 值变化时
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'crontab-hour',

  props: ['check', 'cron'],

  data()
  {
    return {
      radioValue: 1,
      cycle01: 0,
      cycle02: 1,
      average01: 0,
      average02: 1,
      checkboxList: [],
      checkNum: this.$options.propsData.check,
    }
  },

  computed:
  {
    /**
     * 计算两个周期值
     * @returns {String} 周期范围字符串
     */
    cycleTotal()
    {
      const cycle01 = this.checkNum(this.cycle01, 0, 22)
      const cycle02 = this.checkNum(this.cycle02, cycle01 ? cycle01 + 1 : 1, 23)
      return cycle01 + '-' + cycle02
    },

    /**
     * 计算平均用到的值
     * @returns {String} 平均值字符串
     */
    averageTotal()
    {
      const average01 = this.checkNum(this.average01, 0, 22)
      const average02 = this.checkNum(this.average02, 1, 23 - average01 || 0)
      return average01 + '/' + average02
    },

    /**
     * 计算勾选的checkbox值合集
     * @returns {String} 勾选值字符串
     */
    checkboxString()
    {
      let str = this.checkboxList.join()
      return str == '' ? '*' : str
    },
  },

  watch:
  {
    radioValue: 'radioChange',
    cycleTotal: 'cycleChange',
    averageTotal: 'averageChange',
    checkboxString: 'checkboxChange',
  },

  methods:
  {
    /**
     * 单选按钮值变化时
     */
    radioChange()
    {
      if (this.cron.min === '*')
      {
        this.$emit('update', 'min', '0', 'hour')
      }
      if (this.cron.second === '*')
      {
        this.$emit('update', 'second', '0', 'hour')
      }
      switch (this.radioValue)
      {
        case 1:
          this.$emit('update', 'hour', '*')
          break
        case 2:
          this.$emit('update', 'hour', this.cycleTotal)
          break
        case 3:
          this.$emit('update', 'hour', this.averageTotal)
          break
        case 4:
          this.$emit('update', 'hour', this.checkboxString)
          break
      }
    },

    /**
     * 周期两个值变化时
     */
    cycleChange()
    {
      if (this.radioValue == '2')
      {
        this.$emit('update', 'hour', this.cycleTotal)
      }
    },

    /**
     * 平均两个值变化时
     */
    averageChange()
    {
      if (this.radioValue == '3')
      {
        this.$emit('update', 'hour', this.averageTotal)
      }
    },

    /**
     * checkbox值变化时
     */
    checkboxChange()
    {
      if (this.radioValue == '4')
      {
        this.$emit('update', 'hour', this.checkboxString)
      }
    },
  },
}
</script>