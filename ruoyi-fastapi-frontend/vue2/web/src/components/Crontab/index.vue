<style lang="scss" scoped>
.pop_btn { margin-top: 20px; text-align: center; }
.popup-main { position: relative; overflow: hidden; margin: 10px auto; background: #fff; font-size: 12px; border-radius: 5px; }
.popup-title { overflow: hidden; padding-top: 6px; background: #f2f2f2; line-height: 34px; }
.popup-result { position: relative; box-sizing: border-box; margin: 25px auto; padding: 15px 10px 10px; border: 1px solid #ccc; line-height: 24px; }
.popup-result .title { position: absolute; top: -28px; left: 50%; width: 140px; margin-left: -70px; background: #fff; font-size: 14px; text-align: center; line-height: 30px; }
.popup-result table { width: 100%; margin: 0 auto; text-align: center; }
.popup-result table span { display: block; overflow: hidden; width: 100%; height: 30px; border: 1px solid #e8e8e8; font-family: arial; line-height: 30px; white-space: nowrap; }
.popup-result-scroll { overflow-y: auto; height: 10em; font-size: 12px; line-height: 24px; }
</style>

<template>
  <div>
    <el-tabs type="border-card">
      <el-tab-pane label="秒" v-if="shouldHide('second')">
        <CrontabSecond
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronsecond"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="分钟" v-if="shouldHide('min')">
        <CrontabMin
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronmin"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="小时" v-if="shouldHide('hour')">
        <CrontabHour
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronhour"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="日" v-if="shouldHide('day')">
        <CrontabDay
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronday"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="月" v-if="shouldHide('month')">
        <CrontabMonth
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronmonth"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="周" v-if="shouldHide('week')">
        <CrontabWeek
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronweek"
          @update="updateCrontabValue"/>
      </el-tab-pane>

      <el-tab-pane label="年" v-if="shouldHide('year')">
        <CrontabYear
          :check="checkNumber"
          :cron="crontabValueObj"
          ref="cronyear"
          @update="updateCrontabValue"/>
      </el-tab-pane>
    </el-tabs>

    <div class="popup-main">
      <div class="popup-result">
        <p class="title">时间表达式</p>
        <table>
          <thead>
            <th
              v-for="item of tabTitles"
              width="40"
              :key="item">
              {{ item }}
            </th>
            <th>Cron 表达式</th>
          </thead>
          <tbody>
            <td>
              <span>{{ crontabValueObj.second }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.min }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.hour }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.day }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.month }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.week }}</span>
            </td>
            <td>
              <span>{{ crontabValueObj.year }}</span>
            </td>
            <td>
              <span>{{ crontabValueString }}</span>
            </td>
          </tbody>
        </table>
      </div>
      <CrontabResult :ex="crontabValueString" :time-zone="timeZone"></CrontabResult>

      <div class="pop_btn">
        <el-button
          size="small"
          type="primary"
          @click="submitFill">确定</el-button>
        <el-button
          size="small"
          type="warning"
          @click="clearCron">重置</el-button>
        <el-button size="small" @click="hidePopup">取消</el-button>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：Crontab
 * 功能说明：Cron 表达式可视化配置组件，支持秒/分/时/日/月/周/年各字段配置与预览
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} expression    - Cron 表达式
 * @prop {Array}  hideComponent - 需要隐藏的字段数组
 * @prop {String} timeZone      - 时区
 *
 * 变量说明：
 * @data {Array}  tabTitles       - 时间字段标题列表
 * @data {Number} tabActive       - 当前激活的标签索引
 * @data {Number} myindex         - 当前索引
 * @data {Object} crontabValueObj - Cron 表达式各字段值对象
 *
 * 事件说明：
 * @event shouldHide         - 判断指定字段是否需要显示
 * @event resolveExp         - 反解析表达式
 * @event tabCheck           - 标签切换赋值
 * @event updateCrontabValue - 更新表达式组成的字段值
 * @event changeRadio        - 赋值到子组件
 * @event checkNumber        - 校验并限制数字格式
 * @event hidePopup          - 隐藏弹窗
 * @event submitFill         - 填充表达式
 * @event clearCron          - 还原选择项
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import CrontabSecond from './second.vue'
import CrontabMin from './min.vue'
import CrontabHour from './hour.vue'
import CrontabDay from './day.vue'
import CrontabMonth from './month.vue'
import CrontabWeek from './week.vue'
import CrontabYear from './year.vue'
import CrontabResult from './result.vue'

export default
{
  name: 'vcrontab',

  components:
  {
    CrontabSecond,
    CrontabMin,
    CrontabHour,
    CrontabDay,
    CrontabMonth,
    CrontabWeek,
    CrontabYear,
    CrontabResult
  },

  props:
  {
    expression: String,
    hideComponent: Array,
    timeZone: { type: String, required: true }
  },

  data()
  {
    return {
      tabTitles: ['秒', '分钟', '小时', '日', '月', '周', '年'],
      tabActive: 0,
      myindex: 0,
      crontabValueObj: {
        second: '*',
        min: '*',
        hour: '*',
        day: '*',
        month: '*',
        week: '?',
        year: '',
      },
    }
  },

  computed:
  {
    /**
     * 拼接生成 Cron 表达式字符串
     * @returns {String} Cron 表达式字符串
     */
    crontabValueString()
    {
      let obj = this.crontabValueObj
      let str =
        obj.second +
        ' ' +
        obj.min +
        ' ' +
        obj.hour +
        ' ' +
        obj.day +
        ' ' +
        obj.month +
        ' ' +
        obj.week +
        (obj.year == '' ? '' : ' ' + obj.year)
      return str
    },
  },

  watch:
  {
    expression: 'resolveExp',

    /**
     * 隐藏部分组件
     */
    hideComponent(value)
    {
      // 隐藏部分组件
    },
  },

  /**
   * 组件挂载完成后反解析表达式
   */
  mounted()
  {
    this.resolveExp()
  },

  methods:
  {
    /**
     * 判断指定字段是否需要显示
     * @param {String} key - 字段名
     * @returns {Boolean} 是否显示
     */
    shouldHide(key)
    {
      if (this.hideComponent && this.hideComponent.includes(key)) return false
      return true
    },

    /**
     * 反解析表达式
     */
    resolveExp()
    {
      // 反解析 表达式
      if (this.expression)
      {
        let arr = this.expression.split(' ')
        if (arr.length >= 6)
        {
          //6 位以上是合法表达式
          let obj = {
            second: arr[0],
            min: arr[1],
            hour: arr[2],
            day: arr[3],
            month: arr[4],
            week: arr[5],
            year: arr[6] ? arr[6] : '',
          }
          this.crontabValueObj = {
            ...obj,
          }
          for (let i in obj)
          {
            if (obj[i]) this.changeRadio(i, obj[i])
          }
        }
      }
      else
      {
        // 没有传入的表达式 则还原
        this.clearCron()
      }
    },

    /**
     * tab切换值
     * @param {Number} index - 标签索引
     */
    tabCheck(index)
    {
      this.tabActive = index
    },

    /**
     * 由子组件触发，更改表达式组成的字段值
     * @param {String} name - 字段名
     * @param {String} value - 字段值
     * @param {String} from - 来源组件
     */
    updateCrontabValue(name, value, from)
    {
      ;('updateCrontabValue', name, value, from)
      this.crontabValueObj[name] = value
      if (from && from !== name)
      {
        console.log(`来自组件 ${from} 改变了 ${name} ${value}`)
        this.changeRadio(name, value)
      }
    },

    /**
     * 赋值到组件
     * @param {String} name - 字段名
     * @param {String} value - 字段值
     */
    changeRadio(name, value)
    {
      let arr = ['second', 'min', 'hour', 'month'],
        refName = 'cron' + name,
        insValue

      if (!this.$refs[refName]) return

      if (arr.includes(name))
      {
        if (value === '*')
        {
          insValue = 1
        }
        else if (value.indexOf('-') > -1)
        {
          let indexArr = value.split('-')
          isNaN(indexArr[0])
            ? (this.$refs[refName].cycle01 = 0)
            : (this.$refs[refName].cycle01 = indexArr[0])
          this.$refs[refName].cycle02 = indexArr[1]
          insValue = 2
        }
        else if (value.indexOf('/') > -1)
        {
          let indexArr = value.split('/')
          isNaN(indexArr[0])
            ? (this.$refs[refName].average01 = 0)
            : (this.$refs[refName].average01 = indexArr[0])
          this.$refs[refName].average02 = indexArr[1]
          insValue = 3
        }
        else
        {
          insValue = 4
          this.$refs[refName].checkboxList = value.split(',')
        }
      }
      else if (name == 'day')
      {
        if (value === '*')
        {
          insValue = 1
        }
        else if (value == '?')
        {
          insValue = 2
        }
        else if (value.indexOf('-') > -1)
        {
          let indexArr = value.split('-')
          isNaN(indexArr[0])
            ? (this.$refs[refName].cycle01 = 0)
            : (this.$refs[refName].cycle01 = indexArr[0])
          this.$refs[refName].cycle02 = indexArr[1]
          insValue = 3
        }
        else if (value.indexOf('/') > -1)
        {
          let indexArr = value.split('/')
          isNaN(indexArr[0])
            ? (this.$refs[refName].average01 = 0)
            : (this.$refs[refName].average01 = indexArr[0])
          this.$refs[refName].average02 = indexArr[1]
          insValue = 4
        }
        else if (value.indexOf('W') > -1)
        {
          let indexArr = value.split('W')
          isNaN(indexArr[0])
            ? (this.$refs[refName].workday = 0)
            : (this.$refs[refName].workday = indexArr[0])
          insValue = 5
        }
        else if (value === 'L')
        {
          insValue = 6
        }
        else
        {
          this.$refs[refName].checkboxList = value.split(',')
          insValue = 7
        }
      }
      else if (name == 'week')
      {
        if (value === '*')
        {
          insValue = 1
        }
        else if (value == '?')
        {
          insValue = 2
        }
        else if (value.indexOf('-') > -1)
        {
          let indexArr = value.split('-')
          isNaN(indexArr[0])
            ? (this.$refs[refName].cycle01 = 0)
            : (this.$refs[refName].cycle01 = indexArr[0])
          this.$refs[refName].cycle02 = indexArr[1]
          insValue = 3
        }
        else if (value.indexOf('#') > -1)
        {
          let indexArr = value.split('#')
          isNaN(indexArr[0])
            ? (this.$refs[refName].average01 = 1)
            : (this.$refs[refName].average01 = indexArr[0])
          this.$refs[refName].average02 = indexArr[1]
          insValue = 4
        }
        else if (value.indexOf('L') > -1)
        {
          let indexArr = value.split('L')
          isNaN(indexArr[0])
            ? (this.$refs[refName].weekday = 1)
            : (this.$refs[refName].weekday = indexArr[0])
          insValue = 5
        }
        else
        {
          this.$refs[refName].checkboxList = value.split(',')
          insValue = 6
        }
      }
      else if (name == 'year')
      {
        if (value == '')
        {
          insValue = 1
        }
        else if (value == '*')
        {
          insValue = 2
        }
        else if (value.indexOf('-') > -1)
        {
          insValue = 3
        }
        else if (value.indexOf('/') > -1)
        {
          insValue = 4
        }
        else
        {
          this.$refs[refName].checkboxList = value.split(',')
          insValue = 5
        }
      }
      this.$refs[refName].radioValue = insValue
    },

    /**
     * 表单选项的子组件校验数字格式（通过-props传递）
     * @param {Number} value - 待校验的值
     * @param {Number} minLimit - 最小值
     * @param {Number} maxLimit - 最大值
     * @returns {Number} 校验后的值
     */
    checkNumber(value, minLimit, maxLimit)
    {
      // 检查必须为整数
      value = Math.floor(value)
      if (value < minLimit)
      {
        value = minLimit
      }
      else if (value > maxLimit)
      {
        value = maxLimit
      }
      return value
    },

    /**
     * 隐藏弹窗
     */
    hidePopup()
    {
      this.$emit('hide')
    },

    /**
     * 填充表达式
     */
    submitFill()
    {
      this.$emit('fill', this.crontabValueString)
      this.hidePopup()
    },

    /**
     * 还原选择项
     */
    clearCron()
    {
      // 还原选择项
      ;('准备还原')
      this.crontabValueObj = {
        second: '*',
        min: '*',
        hour: '*',
        day: '*',
        month: '*',
        week: '?',
        year: '',
      }
      for (let j in this.crontabValueObj)
      {
        this.changeRadio(j, this.crontabValueObj[j])
      }
    },
  }
}
</script>