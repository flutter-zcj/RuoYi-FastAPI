<style lang="scss" scoped>
.business-datetime-input { width: 100%; line-height: 1.5; }
.business-datetime-input__fields { display: grid; grid-template-columns: minmax(0, 1.15fr) minmax(0, 1fr); gap: 6px; }
.business-datetime-input__zone { margin-top: 3px; color: #606266; font-size: 12px; }
.business-datetime-input__error { color: #c45656; font-size: 12px; }
</style>

<template>
  <div class="business-datetime-input">
    <div class="business-datetime-input__fields">
      <el-input
        v-model="dateInput"
        type="date"
        :aria-label="label + '日期'"
        :disabled="disabled"
        :clearable="clearable"/>
      <el-input
        v-model="timeInput"
        type="time"
        step="1"
        :aria-label="label + '时间'"
        :disabled="disabled"/>
    </div>
    <div class="business-datetime-input__zone">{{ timezoneName }}</div>
    <div
      class="business-datetime-input__error"
      v-if="validationMessage"
      role="alert">
      {{ validationMessage }}
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：BusinessDateTimePicker
 * 功能说明：业务日期时间选择器组件，分别输入日期与时间并校验时区有效性
 * 研发时间：2026/09/12
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String}  value     - 业务时间值
 * @prop {String}  label     - 输入框标签
 * @prop {String}  timezone  - 时区名称
 * @prop {Boolean} disabled  - 是否禁用
 * @prop {Boolean} clearable - 是否可清空
 *
 * 变量说明：
 *
 * 事件说明：
 * @event input - 更新业务时间值
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { formatBusinessTime, getDisplayTimezone, getWallTimeCandidates } from '@/utils/time'

// 日期和时间分别保存为字符串，避免 Date 控件先按宿主机时区修正不存在的时间。
export default
{
  name: 'BusinessDateTimePicker',

  props:
  {
    value: { type: String, default: undefined },
    label: { type: String, default: '日期时间' },
    timezone: { type: String, default: '' },
    disabled: Boolean,
    clearable: { type: Boolean, default: true }
  },

  computed:
  {
    /**
     * 读取原始业务时间值
     * @returns {String} 业务时间值
     */
    wallValue()
    {
      return this.value
    },

    /**
     * 获取显示时区名称
     * @returns {String} 时区名称
     */
    timezoneName()
    {
      return this.timezone || getDisplayTimezone()
    },

    dateInput:
    {
      /**
       * 读取日期部分
       * @returns {String} 日期字符串
       */
      get()
      {
        return this.wallValue?.slice(0, 10) || ''
      },

      /**
       * 设置日期并同步更新业务时间
       * @param {String} value - 日期字符串
       */
      set(value)
      {
        this.updateValue(value ? `${value} ${this.timeInput || '00:00:00'}` : null)
      }
    },

    timeInput:
    {
      /**
       * 读取时间部分
       * @returns {String} 时间字符串
       */
      get()
      {
        return this.wallValue?.slice(11, 19) || ''
      },

      /**
       * 设置时间并同步更新业务时间
       * @param {String} value - 时间字符串
       */
      set(value)
      {
        const date =
          this.dateInput || formatBusinessTime(new Date(), 'YYYY-MM-DD', this.timezoneName)
        const time = value.length === 5 ? value + ':00' : value
        this.updateValue(`${date} ${time}`)
      }
    },

    /**
     * 校验业务时间在当前时区是否有效
     * @returns {String} 校验提示信息
     */
    validationMessage()
    {
      if (!this.wallValue)
      {
        return ''
      }
      try
      {
        return getWallTimeCandidates(this.wallValue, this.timezoneName).length
          ? ''
          : '该时间在此时区不存在，请重新选择'
      } catch {
        return '请选择完整有效的日期和时间'
      }
    }
  },

  methods:
  {
    /**
     * 通过 Vue 2 的 value/input 绑定更新业务时间。
     *
     * @param {string|null} value 业务时间，清空时为 null
     */
    updateValue(value)
    {
      this.$emit('input', value)
    }
  }
}
</script>
