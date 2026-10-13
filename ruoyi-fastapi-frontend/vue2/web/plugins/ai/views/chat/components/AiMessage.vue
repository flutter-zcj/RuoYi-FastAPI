<style lang="scss">
.ai-message-container { width: 100%;
  .ai-message-content { font-size: 14px; line-height: 1.6; color: #606266; overflow-wrap: break-word; word-break: break-word; }
}
.reasoning-section { margin-bottom: 15px; border-left: 2px solid #dcdfe6; padding-left: 10px;
  .reasoning-header { display: flex; align-items: center; cursor: pointer; color: #909399; font-size: 13px; user-select: none; margin-bottom: 5px;
    i { margin-right: 4px; transition: transform 0.3s;
      &.is-expanded { transform: rotate(90deg); }
    }
    .reasoning-status { margin-left: 10px; font-size: 12px; color: #e6a23c; animation: blink 1.5s infinite; }
  }
  .reasoning-content { font-size: 13px; color: #606266; padding: 8px; background-color: #f5f7fa; border-radius: 4px; overflow-wrap: break-word; word-break: break-word; }
}
@keyframes blink {
  0% { opacity: 1; }
  50% { opacity: 0.5; }
  100% { opacity: 1; }
}
.typing-indicator { display: flex; align-items: center; padding: 8px 0;
  span { display: inline-block; width: 6px; height: 6px; background-color: #909399; border-radius: 50%; animation: typing 1.4s infinite ease-in-out both; margin-right: 4px;
    &:nth-child(1) { animation-delay: -0.32s; }
    &:nth-child(2) { animation-delay: -0.16s; }
  }
}
@keyframes typing {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}
</style>

<template>
  <div class="ai-message-container">
    <div class="reasoning-section" v-if="reasoningContent">
      <div class="reasoning-header" @click="toggleReasoning">
        <i :class="['el-icon-arrow-right', { 'is-expanded': isReasoningExpanded }]"></i>
        <span>深度思考过程</span>
        <span class="reasoning-status" v-if="!isThinkingComplete">思考中...</span>
      </div>
      <div class="reasoning-content" v-show="isReasoningExpanded">
        <MarkdownRender
          :content="reasoningContent"
          :is-dark="isDark"
          :final="!loading"
          mode="chat"
          code-renderer="shiki"/>
      </div>
    </div>
    <div class="ai-message-content">
      <MarkdownRender
        :content="content"
        :is-dark="isDark"
        :final="!loading"
        mode="chat"
        code-renderer="shiki"/>
    </div>
    <div class="typing-indicator" v-if="loading && !content && !reasoningContent">
      <span></span>
      <span></span>
      <span></span>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：AiMessage
 * 功能说明：AI 消息内容组件，支持推理过程折叠展示与 Markdown 渲染
 * 研发时间：2026/07/28
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String}  content          - 消息内容
 * @prop {String}  reasoningContent - 推理内容
 * @prop {Boolean} loading          - 是否加载中
 * @prop {Boolean} isDark           - 是否深色主题
 *
 * 变量说明：
 * @data {Boolean} isReasoningExpanded - 是否展开推理内容
 * @data {Boolean} isThinkingComplete  - 推理是否完成
 *
 * 事件说明：
 * @event toggleReasoning - 切换推理内容的展开状态
 *
 * 插槽说明：
 */

/**
 * 2026/07/28     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { defineComponent, ref, computed } from 'vue'

export default defineComponent(
{
  name: 'AiMessage',

  props:
  {
    content: { type: String, default: '' },
    reasoningContent: { type: String, default: '' },
    loading: { type: Boolean, default: false },
    isDark: { type: Boolean, default: false }
  },

  setup(props)
  {
    /**
     * 是否展开推理内容
     */
    const isReasoningExpanded = ref(true)

    /**
     * 推理是否完成
     */
    const isThinkingComplete = computed(() =>
    {
      return !!props.content
    })

    /**
     * 切换推理内容的展开状态
     */
    function toggleReasoning()
    {
      isReasoningExpanded.value = !isReasoningExpanded.value
    }

    return {
      isReasoningExpanded,
      isThinkingComplete,
      toggleReasoning,
    }
  }
})
</script>