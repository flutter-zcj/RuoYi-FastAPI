<style lang="scss" scoped>
@media (max-width: 640px) {
  .secret-form :deep(.el-form-item) { display: block; }
  .secret-form :deep(.el-form-item__label) { width: auto !important; margin-bottom: 6px; }
  .secret-form :deep(.el-form-item__content) { margin-left: 0 !important; }
}
</style>

<template>
  <el-dialog
    custom-class="oauth-overlay"
    :visible.sync="visible"
    title="保存应用密钥"
    width="min(680px, calc(100vw - 48px))"
    append-to-body
    :close-on-click-modal="false">
    <el-form class="secret-form" label-width="160px">
      <el-form-item>
        <template #label><FieldLabel label="应用编号" help="系统生成的应用编号，复制后用于配置外部应用。"/></template>
        <el-input :value="secret.clientId" readonly/>
      </el-form-item>
      <el-form-item>
        <template #label><FieldLabel label="应用密钥" help="应用密钥相当于应用密码，只显示这一次；请复制后保存在安全的密钥管理系统中。"/></template>
        <el-input :value="secret.clientSecret" readonly>
          <template #append>
            <el-button icon="el-icon-document-copy" @click="copySecret">复制</el-button>
          </template>
        </el-input>
      </el-form-item>
    </el-form>
    <el-checkbox v-model="confirmed">我已将密钥保存到安全位置</el-checkbox>
    <template #footer>
      <el-button
        type="primary"
        :disabled="!confirmed"
        @click="close">完成</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import { computed, getCurrentInstance, ref, watch } from 'vue'
import FieldLabel from '@/components/OAuthWorkspace/FieldLabel.vue'

/**
 * 组件名称：OAuthClientSecret
 * 功能说明：应用密钥展示弹窗，一次性展示新生成的客户端密钥并支持复制
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} value  - 弹窗是否显示
 * @prop {Object}  secret - 密钥数据（clientId、clientSecret）
 *
 * 变量说明：
 * @data {Object}  props     - 组件入参
 * @data {Object}  emit      - 组件事件触发器
 * @data {Boolean} confirmed - 是否已确认保存密钥
 * @data {Object}  visible   - 弹窗显示状态（v-model）
 *
 * 事件说明：
 * @event input      - 更新弹窗显示状态
 * @event copySecret - 复制本次生成的客户端密钥
 * @event close      - 关闭密钥展示对话框
 *
 * 插槽说明：
 */

/**
 * 2026/10/01     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

const props = defineProps({
  value: { type: Boolean, default: false },
  secret: { type: Object, default: () => ({}) },
})
const emit = defineEmits(['input'])
const { proxy } = getCurrentInstance()
const confirmed = ref(false)
const visible = computed({
  get: () => props.value,
  set: (value) => emit('input', value),
})

watch(
  () => props.value,
  (value) => {
    if (value) {
      confirmed.value = false
    }
  }
)

/**
 * 复制本次生成的客户端密钥
 */
async function copySecret()
{
  if (!props.secret.clientSecret) {
    return
  }
  try {
    await navigator.clipboard.writeText(props.secret.clientSecret)
    proxy.$modal.msgSuccess('密钥已复制')
  } catch {
    proxy.$modal.msgError('浏览器拒绝访问剪贴板，请手动复制')
  }
}

/**
 * 关闭密钥展示对话框
 */
function close()
{
  visible.value = false
}
</script>