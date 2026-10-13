<style lang="scss" scoped>
.register { display: flex; justify-content: center; align-items: center; height: 100%; background-image: url('../assets/images/login-background.jpg'); background-size: cover; }
.title { margin: 0px auto 30px auto; color: #707070; text-align: center; }
.register-form { border-radius: 6px; background: #ffffff; width: 400px; padding: 25px 25px 5px 25px;
  .el-input { height: 38px;
    input { height: 38px; }
  }
  .input-icon { height: 39px; width: 14px; margin-left: 2px; }
}
.register-tip { font-size: 13px; color: #bfbfbf; text-align: center; }
.register-code { float: right; width: 33%; height: 38px;
  img { vertical-align: middle; cursor: pointer; }
}
.el-register-footer { position: fixed; bottom: 0; height: 40px; width: 100%; color: #fff; font-family: Arial; font-size: 12px; letter-spacing: 1px; line-height: 40px; text-align: center; }
.register-code-img { height: 38px; }
</style>

<template>
  <div class="register">
    <el-form
      class="register-form"
      ref="registerForm"
      :model="registerForm"
      :rules="registerRules">
      <h3 class="title">{{ title }}</h3>
      <el-form-item prop="username">
        <el-input
          v-model="registerForm.username"
          type="text"
          auto-complete="off"
          placeholder="账号">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="user"
            slot="prefix"/>
        </el-input>
      </el-form-item>
      <el-form-item prop="password" :rules="registerPwdValidator">
        <el-input
          v-model="registerForm.password"
          type="password"
          auto-complete="off"
          placeholder="密码"
          @keyup.enter.native="handleRegister">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="password"
            slot="prefix"/>
        </el-input>
      </el-form-item>
      <el-form-item prop="confirmPassword">
        <el-input
          v-model="registerForm.confirmPassword"
          type="password"
          auto-complete="off"
          placeholder="确认密码"
          @keyup.enter.native="handleRegister">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="password"
            slot="prefix"/>
        </el-input>
      </el-form-item>
      <el-form-item v-if="captchaEnabled" prop="code">
        <el-input
          v-model="registerForm.code"
          auto-complete="off"
          placeholder="验证码"
          style="width: 63%"
          @keyup.enter.native="handleRegister">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="validCode"
            slot="prefix"/>
        </el-input>
        <div class="register-code">
          <img
            class="register-code-img"
            :src="codeUrl"
            @click="getCode"/>
        </div>
      </el-form-item>
      <el-form-item style="width: 100%">
        <el-button
          :loading="loading"
          size="medium"
          type="primary"
          style="width: 100%"
          @click.native.prevent="handleRegister">
          <span v-if="!loading">注 册</span>
          <span v-else>注 册 中...</span>
        </el-button>
        <div style="float: right">
          <router-link class="link-type" :to="'/login'">使用已有账户登录</router-link>
        </div>
      </el-form-item>
    </el-form>
    <!--  底部  -->
    <div class="el-register-footer">
      <span>{{ footerContent }}</span>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：Register
 * 功能说明：用户注册页面，校验并提交账号、密码与验证码完成注册
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String}  title          - 页面标题
 * @data {String}  footerContent  - 页脚内容
 * @data {String}  codeUrl        - 验证码图片地址
 * @data {Object}  registerForm   - 注册表单数据（username 账号 / password 密码 / confirmPassword 确认密码 / code 验证码 / uuid 唯一标识）
 * @data {Boolean} loading        - 注册提交加载状态
 * @data {Boolean} captchaEnabled - 是否开启验证码
 *
 * 事件说明：
 * @event getCode        - 获取验证码图片与唯一标识
 * @event handleRegister - 校验并提交注册信息
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getCodeImg, register } from '@/api/login'
import passwordRule from '@/utils/passwordRule'
import defaultSettings from '@/settings'

export default
{
  mixins: [passwordRule],

  data()
  {
    return {
      title: process.env.VUE_APP_TITLE,
      footerContent: defaultSettings.footerContent,
      codeUrl: '',
      registerForm: {
        username: '',
        password: '',
        confirmPassword: '',
        code: '',
        uuid: '',
      },
      loading: false,
      captchaEnabled: true,
    }
  },

  computed:
  {
    /**
     * 计算注册表单校验规则
     * @returns {Object} 注册表单校验规则
     */
    registerRules()
    {
      return {
        username: [
          { required: true, trigger: 'blur', message: '请输入您的账号' },
          { min: 2, max: 20, message: '用户账号长度必须介于 2 和 20 之间', trigger: 'blur' },
        ],
        confirmPassword: [
          { required: true, message: '请再次输入您的密码', trigger: 'blur' },
          {
            validator: (rule, value, callback) => {
              if (this.registerForm.password !== value) {
                callback(new Error('两次输入的密码不一致'))
              } else {
                callback()
              }
            },
            trigger: 'blur',
          },
        ],
        code: [{ required: true, trigger: 'change', message: '请输入验证码' }],
      }
    },
  },

  /**
   * 组件创建时获取验证码
   */
  created()
  {
    this.getCode()
  },

  methods:
  {
    /**
     * 获取验证码图片与唯一标识
     */
    getCode()
    {
      getCodeImg().then((res) => {
        this.captchaEnabled = res.captchaEnabled === undefined ? true : res.captchaEnabled
        if (this.captchaEnabled) {
          this.codeUrl = 'data:image/gif;base64,' + res.img
          this.registerForm.uuid = res.uuid
        }
      })
    },

    /**
     * 校验并提交注册信息
     */
    handleRegister()
    {
      this.$refs.registerForm.validate((valid) => {
        if (valid) {
          this.loading = true
          register(this.registerForm)
            .then(() => {
              const username = this.registerForm.username
              this.$alert(
                "<font color='red'>恭喜你，您的账号 " + username + ' 注册成功！</font>',
                '系统提示',
                {
                  dangerouslyUseHTMLString: true,
                  type: 'success',
                }
              )
                .then(() => {
                  this.$router.push('/login')
                })
                .catch(() => {})
            })
            .catch(() => {
              this.loading = false
              if (this.captchaEnabled) {
                this.getCode()
              }
            })
        }
      })
    },
  },
}
</script>