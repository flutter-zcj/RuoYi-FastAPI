<style lang="scss" scoped></style>

<template>
  <el-form size="small">
    <el-form-item>
      <el-radio :label="1" v-model="radioValue">
        不填，允许的通配符[, - * /]
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio :label="2" v-model="radioValue">
        每年
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio :label="3" v-model="radioValue">
        周期从
        <el-input-number
          v-model="cycle01"
          :min="fullYear"
          :max="2098"/>
        -
        <el-input-number
          v-model="cycle02"
          :min="cycle01 ? cycle01 + 1 : fullYear + 1"
          :max="2099"/>
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio :label="4" v-model="radioValue">
        从
        <el-input-number
          v-model="average01"
          :min="fullYear"
          :max="2098"/>
        年开始，每
        <el-input-number
          v-model="average02"
          :min="1"
          :max="2099 - average01 || fullYear"/>
        年执行一次
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio :label="5" v-model="radioValue">
        指定
        <el-select
          clearable
          v-model="checkboxList"
          placeholder="可多选"
          multiple>
          <el-option
            v-for="item in 9"
            :key="item"
            :value="item - 1 + fullYear"
            :label="item - 1 + fullYear"/>
        </el-select>
      </el-radio>
    </el-form-item>
  </el-form>
</template>

<script>
/**
 * 组件名称：Crontab
 * 功能说明：Cron 表达式年字段配置组件
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Function} check - 数字格式校验方法
 * @prop {Object}   month - 月份数据对象
 * @prop {Object}   cron  - Cron 表达式对象
 *
 * 变量说明：
 * @data {Number}   fullYear     - 当前年份
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
  name: 'crontab-year',

  props: ['check', 'month', 'cron'],

  data()
  {
    return {
      fullYear: 0,
      radioValue: 1,
      cycle01: 0,
      cycle02: 0,
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
      const cycle01 = this.checkNum(this.cycle01, this.fullYear, 2098)
      const cycle02 = this.checkNum(this.cycle02, cycle01 ? cycle01 + 1 : this.fullYear + 1, 2099)
      return cycle01 + '-' + cycle02
    },

    /**
     * 计算平均用到的值
     * @returns {String} 平均值字符串
     */
    averageTotal()
    {
      const average01 = this.checkNum(this.average01, this.fullYear, 2098)
      const average02 = this.checkNum(this.average02, 1, 2099 - average01 || this.fullYear)
      return average01 + '/' + average02
    },

    /**
     * 计算勾选的checkbox值合集
     * @returns {String} 勾选值字符串
     */
    checkboxString()
    {
      let str = this.checkboxList.join()
      return str
    },
  },

  /**
   * 组件挂载完成后获取当前年份
   */
  mounted()
  {
    // 仅获取当前年份
    this.fullYear = Number(new Date().getFullYear())
    this.cycle01 = this.fullYear
    this.average01 = this.fullYear
  },

  methods:
  {
    /**
     * 单选按钮值变化时
     */
    radioChange()
    {
      switch (this.radioValue)
      {
        case 1:
          this.$emit('update', 'year', '')
          break
        case 2:
          this.$emit('update', 'year', '*')
          break
        case 3:
          this.$emit('update', 'year', this.cycleTotal)
          break
        case 4:
          this.$emit('update', 'year', this.averageTotal)
          break
        case 5:
          this.$emit('update', 'year', this.checkboxString)
          break
      }
    },

    /**
     * 周期两个值变化时
     */
    cycleChange()
    {
      if (this.radioValue == '3')
      {
        this.$emit('update', 'year', this.cycleTotal)
      }
    },

    /**
     * 平均两个值变化时
     */
    averageChange()
    {
      if (this.radioValue == '4')
      {
        this.$emit('update', 'year', this.averageTotal)
      }
    },

    /**
     * checkbox值变化时
     */
    checkboxChange()
    {
      if (this.radioValue == '5')
      {
        this.$emit('update', 'year', this.checkboxString)
      }
    },
  },
}
</script>