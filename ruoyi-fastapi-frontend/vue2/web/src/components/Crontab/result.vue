<style lang="scss" scoped>
.preview-error { color: #f56c6c; }
</style>

<template>
  <div class="popup-result" :aria-busy="loading">
    <p class="title">最近5次运行时间（{{ resultTimeZone }}）</p>
    <ul class="popup-result-scroll" aria-live="polite">
      <li
        class="preview-error"
        v-if="errorMessage"
        role="alert">
        {{ errorMessage }}
      </li>
      <li v-else-if="loading">计算结果中...</li>
      <template v-else>
        <li v-for="item in resultList" :key="item">
          {{ formatBusinessTime(item, 'YYYY-MM-DD HH:mm:ss Z', resultTimeZone) }}
        </li>
        <li v-if="!resultList.length">没有未来执行时间</li>
      </template>
    </ul>
  </div>
</template>

<script>
/**
 * 组件名称：Crontab
 * 功能说明：Cron 表达式最近运行时间预览组件
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} ex       - Cron 表达式
 * @prop {String} timeZone - 时区
 *
 * 变量说明：
 * @data {Array}   resultList     - 预览结果时间列表
 * @data {String}  resultTimeZone - 结果时区
 * @data {Boolean} loading        - 是否正在加载
 * @data {String}  errorMessage   - 错误提示信息
 *
 * 事件说明：
 * @event cancelPreview      - 取消防抖计时和请求
 * @event loadPreview        - 表达式或时区变化后重新预览
 * @event formatBusinessTime - 格式化业务时间
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { previewJob } from '@/api/monitor/job'
import { formatBusinessTime } from '@/utils/time'

export default
{
  name: 'crontab-result',

  props:
  {
    ex: { type: String, default: '' },
    timeZone: { type: String, required: true }
  },

  data()
  {
    return {
      resultList: [],
      resultTimeZone: this.timeZone,
      loading: false,
      errorMessage: '',
    }
  },

  computed:
  {
    /**
     * 预览输入参数
     * @returns {Array} 表达式与时区数组
     */
    previewInput()
    {
      return [this.ex, this.timeZone]
    },
  },

  watch:
  {
    previewInput:
    {
      handler: 'loadPreview',
      immediate: true,
    },
  },

  /**
   * 组件创建前初始化预览相关内部状态
   */
  beforeCreate()
  {
    this._previewTimer = null
    this._previewController = null
    this._previewRequestSequence = 0
  },

  /**
   * 组件销毁前取消预览请求
   */
  beforeDestroy()
  {
    this.cancelPreview()
  },

  methods:
  {
    /**
     * 格式化业务时间
     */
    formatBusinessTime,

    /**
     * 取消防抖计时和请求，让过期响应不再更新组件。
     */
    cancelPreview()
    {
      clearTimeout(this._previewTimer)
      if (this._previewController) this._previewController.abort()
      this._previewRequestSequence++
    },

    /**
     * 表达式或时区变化后重新预览，销毁时释放请求。
     * @param {Array} payload - 表达式与时区数组
     */
    loadPreview([expression, timeZone])
    {
      this.cancelPreview()
      const requestId = this._previewRequestSequence
      const controller = new AbortController()
      this._previewController = controller
      this.loading = true
      this.errorMessage = ''
      this.resultList = []
      this.resultTimeZone = timeZone
      this._previewTimer = setTimeout(async () =>
      {
        try
        {
          const response = await previewJob(
            { cronExpression: expression, timeZone, count: 5 },
            controller.signal
          )
          if (requestId !== this._previewRequestSequence) return
          this.resultList = response.data.nextRunTimes
          this.resultTimeZone = response.data.timeZone
        }
        catch (error)
        {
          if (requestId !== this._previewRequestSequence || controller.signal.aborted) return
          this.errorMessage = error.message || '无法获取运行时间，请稍后重试'
        }
        finally
        {
          if (requestId === this._previewRequestSequence) this.loading = false
        }
      }, 300)
    },
  },
}
</script>