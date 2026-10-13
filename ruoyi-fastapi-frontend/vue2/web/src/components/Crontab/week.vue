<style lang="scss" scoped></style>

<template>
  <el-form size="small">
    <el-form-item>
      <el-radio v-model="radioValue" :label="1">
        周，允许的通配符[, - * ? / L #]
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="2">
        不指定
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="3">
        周期从星期
        <el-select
          clearable
          v-model="cycle01">
          <el-option
            v-for="(item, index) of weekList"
            :key="index"
            :label="item.value"
            :value="item.key"
            :disabled="item.key === 1">{{ item.value }}</el-option>
        </el-select>
        -
        <el-select
          clearable
          v-model="cycle02">
          <el-option
            v-for="(item, index) of weekList"
            :key="index"
            :label="item.value"
            :value="item.key"
            :disabled="item.key < cycle01 && item.key !== 1">{{ item.value }}</el-option>
        </el-select>
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="4">
        第
        <el-input-number
          v-model="average01"
          :min="1"
          :max="4"/>
        周的星期
        <el-select
          clearable
          v-model="average02">
          <el-option
            v-for="(item, index) of weekList"
            :key="index"
            :label="item.value"
            :value="item.key">{{ item.value }}</el-option>
        </el-select>
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="5">
        本月最后一个星期
        <el-select
          clearable
          v-model="weekday">
          <el-option
            v-for="(item, index) of weekList"
            :key="index"
            :label="item.value"
            :value="item.key">{{ item.value }}</el-option>
        </el-select>
      </el-radio>
    </el-form-item>

    <el-form-item>
      <el-radio v-model="radioValue" :label="6">
        指定
        <el-select
          clearable
          v-model="checkboxList"
          placeholder="可多选"
          multiple
          style="width: 100%">
          <el-option
            v-for="(item, index) of weekList"
            :key="index"
            :label="item.value"
            :value="String(item.key)">{{ item.value }}</el-option>
        </el-select>
      </el-radio>
    </el-form-item>
  </el-form>
</template>

<script>
/**
 * 组件名称：Crontab
 * 功能说明：Cron 表达式周字段配置组件
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Function} check - 数字格式校验方法
 * @prop {Object}   cron  - Cron 表达式对象
 *
 * 变量说明：
 * @data {Number}   radioValue   - 当前选中的单选值
 * @data {Number}   weekday      - 最近工作日
 * @data {Number}   cycle01      - 周期起始值
 * @data {Number}   cycle02      - 周期结束值
 * @data {Number}   average01    - 平均起始值
 * @data {Number}   average02    - 平均间隔值
 * @data {Array}    checkboxList - 指定勾选值列表
 * @data {Array}    weekList     - 星期选项列表
 * @data {Function} checkNum     - 数字校验方法
 *
 * 事件说明：
 * @event radioChange    - 单选按钮值变化时
 * @event cycleChange    - 周期两个值变化时
 * @event averageChange  - 平均两个值变化时
 * @event weekdayChange  - 最近工作日值变化时
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
  name: 'crontab-week',

  props: ['check', 'cron'],

  data()
  {
    return {
      radioValue: 2,
      weekday: 2,
      cycle01: 2,
      cycle02: 3,
      average01: 1,
      average02: 2,
      checkboxList: [],
      weekList: [
        {
          key: 2,
          value: '星期一',
        },
        {
          key: 3,
          value: '星期二',
        },
        {
          key: 4,
          value: '星期三',
        },
        {
          key: 5,
          value: '星期四',
        },
        {
          key: 6,
          value: '星期五',
        },
        {
          key: 7,
          value: '星期六',
        },
        {
          key: 1,
          value: '星期日',
        },
      ],
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
      this.cycle01 = this.checkNum(this.cycle01, 1, 7)
      this.cycle02 = this.checkNum(this.cycle02, 1, 7)
      return this.cycle01 + '-' + this.cycle02
    },

    /**
     * 计算平均用到的值
     * @returns {String} 平均值字符串
     */
    averageTotal()
    {
      this.average01 = this.checkNum(this.average01, 1, 4)
      this.average02 = this.checkNum(this.average02, 1, 7)
      return this.average02 + '#' + this.average01
    },

    /**
     * 最近的工作日（格式）
     * @returns {Number} 最近工作日
     */
    weekdayCheck()
    {
      this.weekday = this.checkNum(this.weekday, 1, 7)
      return this.weekday
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
    weekdayCheck: 'weekdayChange',
    checkboxString: 'checkboxChange',
  },

  methods:
  {
    /**
     * 单选按钮值变化时
     */
    radioChange()
    {
      if (this.radioValue !== 2 && this.cron.day !== '?')
      {
        this.$emit('update', 'day', '?', 'week')
      }
      switch (this.radioValue)
      {
        case 1:
          this.$emit('update', 'week', '*')
          break
        case 2:
          this.$emit('update', 'week', '?')
          break
        case 3:
          this.$emit('update', 'week', this.cycleTotal)
          break
        case 4:
          this.$emit('update', 'week', this.averageTotal)
          break
        case 5:
          this.$emit('update', 'week', this.weekdayCheck + 'L')
          break
        case 6:
          this.$emit('update', 'week', this.checkboxString)
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
        this.$emit('update', 'week', this.cycleTotal)
      }
    },

    /**
     * 平均两个值变化时
     */
    averageChange()
    {
      if (this.radioValue == '4')
      {
        this.$emit('update', 'week', this.averageTotal)
      }
    },

    /**
     * 最近工作日值变化时
     */
    weekdayChange()
    {
      if (this.radioValue == '5')
      {
        this.$emit('update', 'week', this.weekday + 'L')
      }
    },

    /**
     * checkbox值变化时
     */
    checkboxChange()
    {
      if (this.radioValue == '6')
      {
        this.$emit('update', 'week', this.checkboxString)
      }
    },
  },
}
</script>