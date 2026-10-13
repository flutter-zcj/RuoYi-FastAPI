<style lang="scss" scoped>
.auth-flow { min-height: 390px; }
.panel-kicker { margin: 0 0 8px; color: var(--oauth-color-primary); font-size: 12px; font-weight: 700; letter-spacing: 0.1em; }
h2 { margin: 0; color: var(--oauth-text-color-primary); font-size: 25px; font-weight: 650; line-height: 1.4; }
.panel-lead { margin: 12px 0 24px; color: var(--oauth-text-color-regular); font-size: 13px; line-height: 1.7; }
.panel-lead strong { color: var(--oauth-text-color-primary); }
.el-alert { margin-bottom: 18px; }
.captcha-row { display: grid; grid-template-columns: minmax(0, 1fr) 132px; gap: 10px; width: 100%; }
.captcha-image { overflow: hidden; height: 40px; padding: 0; background: var(--oauth-fill-color-light); border: 1px solid var(--oauth-border-color); border-radius: var(--oauth-border-radius-base); color: var(--oauth-text-color-regular); }
.captcha-image img { width: 100%; height: 100%; object-fit: cover; }
.preference-row { display: flex; align-items: center; justify-content: space-between; gap: 10px; margin-top: 2px; }
.preference-row small { color: var(--oauth-text-color-secondary); font-size: 11px; }
.primary-action { width: 100%; height: 44px; margin-top: 22px; }
.cancel-action { display: block; margin: 14px auto 0; color: var(--oauth-text-color-regular); }
@media (max-width: 440px) {
  .captcha-row { grid-template-columns: minmax(0, 1fr) 112px; }
  .preference-row { display: block; }
  .preference-row small { display: block; margin: 2px 0 0 24px; }
}
</style>

<template>
  <AuthCenterShell
    :current-step="0"
    :expires-in="interaction.expiresIn"
    :application-name="interaction.client?.clientName"
    :application-id="interaction.client?.clientId">
    <div class="auth-flow" v-loading="initializing">
      <p class="panel-kicker">验证身份</p>
      <h2>使用你的账户继续</h2>
      <p class="panel-lead">
        登录成功后，你将返回
        <strong>{{ applicationName }}</strong>
        完成访问。本次登录不会改变管理后台的登录状态。
      </p>

      <el-alert
        v-if="errorMessage"
        :title="errorMessage"
        type="error"
        show-icon
        :closable="false"/>
      <el-form
        ref="loginRef"
        :model="form"
        :rules="rules"
        label-position="top"
        @submit.native.prevent="submitLogin">
        <el-form-item label="账号" prop="userName">
          <el-input
            v-model="form.userName"
            size="medium"
            autocomplete="username"
            placeholder="请输入你的账号"
            autofocus>
            <template #prefix><i class="el-icon el-icon-user" aria-hidden="true"></i></template>
          </el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input
            v-model="form.password"
            size="medium"
            type="password"
            show-password
            autocomplete="current-password"
            placeholder="请输入登录密码">
            <template #prefix><i class="el-icon el-icon-lock" aria-hidden="true"></i></template>
          </el-input>
        </el-form-item>
        <el-form-item
          v-if="captcha.captchaEnabled"
          label="安全验证码"
          prop="code">
          <div class="captcha-row">
            <el-input
              v-model="form.code"
              size="medium"
              autocomplete="off"
              placeholder="输入图中字符"/>
            <el-button
              class="captcha-image"
              type="text"
              aria-label="看不清，换一张验证码"
              title="看不清，换一张"
              @click="loadCaptcha">
              <img
                v-if="captcha.img"
                :src="`data:image/gif;base64,${captcha.img}`"
                alt="安全验证码"/>
              <span v-else>换一张</span>
            </el-button>
          </div>
        </el-form-item>

        <div class="preference-row">
          <el-checkbox v-model="form.rememberMe">在这台设备上保持登录</el-checkbox>
          <small>仅在你信任的个人设备上勾选</small>
        </div>
        <el-button
          class="primary-action"
          type="primary"
          size="medium"
          native-type="submit"
          :loading="submitting">登录并继续</el-button>
      </el-form>
      <el-button
        class="cancel-action"
        type="text"
        :disabled="submitting"
        @click="cancel">取消并返回应用</el-button>
    </div>
  </AuthCenterShell>
</template>

<script>
/**
 * 组件名称：AuthCenterLogin
 * 功能说明：统一认证中心登录页面，校验账号密码与验证码并提交登录
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {String}   interactionId         - 认证交互标识
 * @data {Function} csrfToken             - 获取 CSRF 令牌
 * @data {Function} goToAction            - 跳转到指定认证动作
 * @data {Function} followServerRedirect  - 跟随服务端重定向
 * @data {Object}   router                - 路由实例
 * @data {Object}   loginRef              - 登录表单引用
 * @data {Boolean}  initializing          - 是否初始化中
 * @data {Boolean}  submitting            - 是否提交中
 * @data {String}   errorMessage          - 错误提示信息
 * @data {Object}   interaction           - 认证交互数据
 * @data {Object}   captcha               - 验证码数据
 * @data {Object}   form                  - 登录表单数据
 * @data {String}   applicationName       - 应用名称
 * @data {Object}   rules                 - 表单校验规则
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/01     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'AuthCenterLogin'
}
</script>

<script setup>
import { computed, getCurrentInstance, reactive, ref } from 'vue'
import {
  cancelInteraction,
  getInteraction,
  getInteractionCaptcha,
  submitInteractionLogin,
} from '@/api/authCenter'
import AuthCenterShell from './AuthCenterShell.vue'
import { useInteractionContext } from './useInteraction'

const { interactionId, csrfToken, goToAction, followServerRedirect } = useInteractionContext()
const router = getCurrentInstance().proxy.$router
const loginRef = ref()
const initializing = ref(true)
const submitting = ref(false)
const errorMessage = ref('')
const interaction = ref({})
const captcha = ref({ captchaEnabled: false, uuid: '', img: '' })
const form = reactive({
  userName: '',
  password: '',
  code: '',
  uuid: '',
  rememberMe: false,
})
const applicationName = computed(() => interaction.value.client?.clientName || '发起登录的应用')
const rules = {
  userName: [{ required: true, message: '请输入账号', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
  code: [
    {
      validator: (_rule, value, callback) =>
        captcha.value.captchaEnabled && !value
          ? callback(new Error('请输入安全验证码'))
          : callback(),
      trigger: 'blur',
    },
  ],
}

/**
 * 读取认证请求并初始化登录状态
 */
async function initialize()
{
  if (!interactionId.value || !csrfToken()) {
    return showFatal('认证请求不完整，请返回应用重新登录')
  }
  try {
    const response = await getInteraction(interactionId.value, csrfToken())
    interaction.value = response.data || {}
    if (interaction.value.nextAction && interaction.value.nextAction !== 'login') {
      return await goToAction(interaction.value.nextAction)
    }
    if (interaction.value.captchaEnabled) {
      await loadCaptcha()
    }
  } catch (error) {
    showFatal(error?.message || '认证请求已失效，请返回应用重新登录')
  } finally {
    initializing.value = false
  }
}

/**
 * 加载认证中心验证码
 */
async function loadCaptcha()
{
  const response = await getInteractionCaptcha(interactionId.value)
  captcha.value = response.data || { captchaEnabled: false }
  form.uuid = captcha.value.uuid || ''
  form.code = ''
}

/**
 * 校验并提交登录凭据
 */
async function submitLogin()
{
  if (!(await loginRef.value?.validate().catch(() => false))) {
    return
  }
  submitting.value = true
  errorMessage.value = ''
  try {
    const response = await submitInteractionLogin(interactionId.value, csrfToken(), { ...form })
    form.password = ''
    const result = response.data || {}
    if (result.nextAction === 'redirect') {
      return followServerRedirect(result.redirectUrl)
    }
    await goToAction(result.nextAction)
  } catch (error) {
    form.password = ''
    errorMessage.value = error?.message || '身份验证失败，请检查账号和密码后重试'
    if (captcha.value.captchaEnabled) {
      await loadCaptcha().catch(() => {})
    }
  } finally {
    submitting.value = false
  }
}

/**
 * 取消当前认证请求
 */
async function cancel()
{
  try {
    const response = await cancelInteraction(interactionId.value, csrfToken())
    if (response.data?.redirectUrl) {
      return followServerRedirect(response.data.redirectUrl)
    }
  } catch (error) {
    errorMessage.value = error?.message || '暂时无法取消认证请求'
  }
}

/**
 * 跳转到认证错误页面
 * @param {String} message - 错误提示信息
 */
function showFatal(message)
{
  router.replace({ path: '/auth-center/error', query: { message } })
}
initialize()
</script>