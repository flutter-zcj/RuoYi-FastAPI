<style lang="scss" scoped>
.login { display: flex; justify-content: center; align-items: center; height: 100%; background-image: url('../assets/images/login-background.jpg'); background-size: cover; }
.title { margin: 0px auto 30px auto; color: #707070; text-align: center; }
.login-form { z-index: 1; border-radius: 6px; background: #ffffff; width: 400px; padding: 25px 25px 5px 25px;
  .el-input { height: 38px;
    input { height: 38px; }
  }
  .input-icon { height: 39px; width: 14px; margin-left: 2px; }
}
.login-tip { font-size: 13px; color: #bfbfbf; text-align: center; }
.login-code { float: right; width: 33%; height: 38px;
  img { vertical-align: middle; cursor: pointer; }
}
.el-login-footer { position: fixed; bottom: 0; height: 40px; width: 100%; color: #fff; font-family: Arial; font-size: 12px; letter-spacing: 1px; line-height: 40px; text-align: center; }
.login-code-img { height: 38px; }
</style>

<template>
  <div class="login">
    <el-form
      class="login-form"
      ref="loginForm"
      :model="loginForm"
      :rules="loginRules">
      <h3 class="title">{{ title }}</h3>
      <el-form-item prop="username">
        <el-input
          v-model="loginForm.username"
          type="text"
          auto-complete="off"
          placeholder="账号">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="user"
            slot="prefix"/>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          v-model="loginForm.password"
          type="password"
          auto-complete="off"
          placeholder="密码"
          @keyup.enter.native="handleLogin">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="password"
            slot="prefix"/>
        </el-input>
      </el-form-item>
      <el-form-item v-if="captchaEnabled" prop="code">
        <el-input
          v-model="loginForm.code"
          auto-complete="off"
          placeholder="验证码"
          style="width: 63%"
          @keyup.enter.native="handleLogin">
          <svg-icon
            class="el-input__icon input-icon"
            icon-class="validCode"
            slot="prefix"/>
        </el-input>
        <div class="login-code">
          <img
            class="login-code-img"
            :src="codeUrl"
            @click="getCode"/>
        </div>
      </el-form-item>
      <el-checkbox v-model="loginForm.rememberMe" style="margin: 0px 0px 25px 0px">记住密码</el-checkbox>
      <el-form-item style="width: 100%">
        <el-button
          :loading="loading"
          size="medium"
          type="primary"
          style="width: 100%"
          @click.native.prevent="handleLogin">
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
        <div v-if="register" style="float: right">
          <router-link class="link-type" :to="'/register'">立即注册</router-link>
        </div>
      </el-form-item>
    </el-form>
    <!--  底部  -->
    <div class="el-login-footer">
      <span>{{ footerContent }}</span>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：Login
 * 功能说明：用户登录页面，校验账号密码与验证码并提交登录
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String}  title          - 页面标题
 * @data {String}  footerContent  - 页脚内容
 * @data {String}  codeUrl        - 验证码图片地址
 * @data {Object}  loginForm      - 登录表单数据（username 账号 / password 密码 / rememberMe 记住密码 / code 验证码 / uuid 唯一标识）
 * @data {Object}  loginRules     - 登录表单校验规则
 * @data {Boolean} loading        - 登录提交加载状态
 * @data {Boolean} captchaEnabled - 是否开启验证码
 * @data {Boolean} register       - 是否开启注册入口
 * @data {String}  redirect       - 登录成功后的重定向地址
 *
 * 事件说明：
 * @event getCode     - 获取验证码图片与唯一标识
 * @event getCookie   - 读取记住密码的 Cookie 信息
 * @event handleLogin - 校验并提交登录信息
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getCodeImg } from '@/api/login'
import Cookies from 'js-cookie'
import { encrypt, decrypt } from '@/utils/jsencrypt'
import defaultSettings from '@/settings'

export default
{
  name: 'Login',

  data()
  {
    return {
      title: process.env.VUE_APP_TITLE,
      footerContent: defaultSettings.footerContent,
      codeUrl: '',
      loginForm: {
        username: '',
        password: '',
        rememberMe: false,
        code: '',
        uuid: '',
      },
      loginRules: {
        username: [{ required: true, trigger: 'blur', message: '请输入您的账号' }],
        password: [{ required: true, trigger: 'blur', message: '请输入您的密码' }],
        code: [{ required: true, trigger: 'change', message: '请输入验证码' }],
      },
      loading: false,
      // 验证码开关
      captchaEnabled: true,
      // 注册开关
      register: false,
      redirect: undefined,
    }
  },

  watch:
  {
    /**
     * 监听路由变化，记录重定向地址
     * @param {Object} route - 路由对象
     */
    $route: {
      handler: function (route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true,
    },
  },

  /**
   * 组件创建时获取验证码并读取记住密码信息
   */
  created()
  {
    this.getCode()
    this.getCookie()
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
        this.register = res.registerEnabled === undefined ? false : res.registerEnabled
        if (this.captchaEnabled) {
          this.codeUrl = 'data:image/gif;base64,' + res.img
          this.loginForm.uuid = res.uuid
        }
      })
    },

    /**
     * 读取记住密码的 Cookie 信息
     */
    getCookie()
    {
      const username = Cookies.get('username')
      const password = Cookies.get('password')
      const rememberMe = Cookies.get('rememberMe')
      this.loginForm = {
        username: username === undefined ? this.loginForm.username : username,
        password: password === undefined ? this.loginForm.password : decrypt(password),
        rememberMe: rememberMe === undefined ? false : Boolean(rememberMe),
      }
    },

    /**
     * 校验并提交登录信息
     */
    handleLogin()
    {
      this.$refs.loginForm.validate((valid) => {
        if (valid) {
          this.loading = true
          if (this.loginForm.rememberMe) {
            Cookies.set('username', this.loginForm.username, { expires: 30 })
            Cookies.set('password', encrypt(this.loginForm.password), { expires: 30 })
            Cookies.set('rememberMe', this.loginForm.rememberMe, { expires: 30 })
          } else {
            Cookies.remove('username')
            Cookies.remove('password')
            Cookies.remove('rememberMe')
          }
          this.$store
            .dispatch('Login', this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || '/' }).catch(() => {})
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