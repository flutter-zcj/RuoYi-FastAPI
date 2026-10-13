<style lang="scss" scoped>
.lock-container { position: fixed; display: flex; flex-direction: column; align-items: center; justify-content: center; inset: 0; z-index: 9999; overflow: hidden; background: linear-gradient(135deg, #0f0c29, #302b63, #24243e); font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif; }
.particle-bg { position: absolute; inset: 0; z-index: 0; }
.lock-time { position: relative; z-index: 1; margin-bottom: 8px; font-size: 72px; font-weight: 200; font-variant-numeric: tabular-nums; color: #fff; letter-spacing: 4px; text-shadow: 0 0 40px rgba(255, 255, 255, 0.3); }
.lock-date { position: relative; z-index: 1; margin-bottom: 48px; font-size: 15px; color: rgba(255, 255, 255, 0.6); letter-spacing: 2px; }
.lock-card { position: relative; z-index: 1; display: flex; flex-direction: column; align-items: center; width: 360px; padding: 40px 48px; border: 1px solid rgba(255, 255, 255, 0.15); border-radius: 24px; background: rgba(255, 255, 255, 0.08); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px); box-shadow: 0 25px 60px rgba(0, 0, 0, 0.4); }
.avatar-wrap { position: relative; margin-bottom: 16px; }
.lock-avatar { display: block; width: 80px; height: 80px; border: 3px solid rgba(255, 255, 255, 0.3); border-radius: 50%; object-fit: cover; }
.lock-icon { position: absolute; bottom: -4px; right: -4px; display: flex; align-items: center; justify-content: center; width: 26px; height: 26px; border-radius: 50%; background: rgba(255, 255, 255, 0.15); font-size: 13px; backdrop-filter: blur(8px); }
.lock-username { margin-bottom: 6px; color: #fff; font-size: 18px; font-weight: 600; letter-spacing: 1px; }
.lock-hint { margin-bottom: 28px; color: rgba(255, 255, 255, 0.5); font-size: 13px; }
.input-wrap { display: flex; align-items: center; width: 100%; padding: 4px 4px 4px 20px; border: 1px solid rgba(255, 255, 255, 0.2); border-radius: 50px; background: rgba(255, 255, 255, 0.1); transition: border-color 0.3s; }
.input-wrap:focus-within { border-color: rgba(255, 255, 255, 0.6); background: rgba(255, 255, 255, 0.13); }
.input-wrap.shake { animation: shake 0.5s ease; }
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-8px); }
  40% { transform: translateX(8px); }
  60% { transform: translateX(-6px); }
  80% { transform: translateX(6px); }
}
.lock-input { flex: 1; padding: 10px 0; border: none; background: transparent; color: #fff; font-size: 15px; outline: none; }
.lock-input::placeholder { color: rgba(255, 255, 255, 0.35); }
.unlock-btn { display: flex; align-items: center; justify-content: center; flex-shrink: 0; width: 42px; height: 42px; border: none; border-radius: 50%; background: linear-gradient(135deg, #667eea, #764ba2); color: #fff; font-size: 18px; cursor: pointer; transition: transform 0.2s, opacity 0.2s; }
.unlock-btn:hover:not(:disabled) { transform: scale(1.08); }
.unlock-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.loading-dot { font-size: 13px; letter-spacing: 1px; }
.error-msg { margin-top: 14px; color: #ff7675; font-size: 13px; text-align: center; animation: fadeIn 0.3s ease; }
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}
.lock-footer { margin-top: 24px; }
.lock-footer a { color: rgba(255, 255, 255, 0.4); font-size: 13px; text-decoration: none; transition: color 0.2s; }
.lock-footer a:hover { color: rgba(255, 255, 255, 0.8); }
</style>

<template>
  <div class="lock-container">
    <!-- 动态粒子背景 -->
    <canvas class="particle-bg" ref="particleCanvas"></canvas>

    <!-- 时钟 -->
    <div class="lock-time">{{ currentTime }}</div>
    <div class="lock-date">{{ currentDate }}</div>

    <!-- 锁屏卡片 -->
    <div class="lock-card">
      <div class="avatar-wrap">
        <img
          class="lock-avatar"
          :src="avatar"
          @error="onAvatarError"/>
        <div class="lock-icon">🔒</div>
      </div>
      <div class="lock-username">{{ nickName }}</div>
      <div class="lock-hint">系统已锁定，请输入密码解锁</div>

      <div class="input-wrap" :class="{ shake: isShaking }">
        <input
          class="lock-input"
          ref="passwordInput"
          v-model="password"
          type="password"
          placeholder="请输入登录密码"
          autocomplete="off"
          @keydown.enter="handleUnlock"/>
        <button
          class="unlock-btn"
          :disabled="loading"
          @click="handleUnlock">
          <span v-if="!loading">→</span>
          <span v-else class="loading-dot">···</span>
        </button>
      </div>

      <div v-if="errorMsg" class="error-msg">
        {{ errorMsg }}
      </div>

      <div class="lock-footer">
        <a href="/login" @click.prevent="goLogin">退出重新登录</a>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：LockScreen
 * 功能说明：系统锁屏页面，展示时钟与粒子背景，校验密码后解锁
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String}   password    - 解锁密码
 * @data {Boolean}  loading     - 解锁提交加载状态
 * @data {String}   errorMsg    - 错误提示信息
 * @data {Boolean}  isShaking   - 是否处于抖动动画状态
 * @data {String}   currentTime - 当前时间
 * @data {String}   currentDate - 当前日期
 * @data {Number}   timer       - 时钟定时器
 * @data {Number}   animationId - 粒子动画帧标识
 * @data {Array}    particles   - 粒子集合
 *
 * 事件说明：
 * @event onAvatarError - 头像加载失败时使用默认头像
 * @event startClock    - 启动时钟定时更新
 * @event handleUnlock  - 校验密码并解锁系统
 * @event showError     - 展示错误提示与抖动动画
 * @event goLogin       - 退出登录并跳转登录页
 * @event initParticles - 初始化粒子背景动画
 *
 * 插槽说明：
 */

/**
 * 2026/07/31     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { formatBusinessTime } from '@/utils/time'
import { mapGetters } from 'vuex'
import { unlockScreen } from '@/api/login'
import defAva from '@/assets/images/profile.jpg'

export default
{
  name: 'LockScreen',

  data()
  {
    return {
      password: '',
      loading: false,
      errorMsg: '',
      isShaking: false,
      currentTime: '',
      currentDate: '',
      timer: null,
      animationId: null,
      particles: [],
    }
  },

  computed:
  {
    ...mapGetters(['avatar', 'nickName']),
  },

  /**
   * 组件挂载完成后启动时钟与粒子背景，并聚焦密码输入框
   */
  mounted()
  {
    this.startClock()
    this.initParticles()
    this.$nextTick(() => {
      this.$refs.passwordInput && this.$refs.passwordInput.focus()
    })
  },

  /**
   * 组件销毁前清理时钟定时器与粒子动画
   */
  beforeDestroy()
  {
    clearInterval(this.timer)
    cancelAnimationFrame(this.animationId)
  },

  methods:
  {
    /**
     * 头像加载失败时使用默认头像
     * @param {Event} e - 图片加载事件对象
     */
    onAvatarError(e)
    {
      e.target.src = defAva
    },

    /**
     * 启动时钟定时更新
     */
    startClock()
    {
      const update = () => {
        const now = new Date()
        this.currentTime = formatBusinessTime(now, 'HH:mm:ss')
        this.currentDate = formatBusinessTime(now, 'YYYY年M月D日 星期{a}')
      }
      update()
      this.timer = setInterval(update, 1000)
    },

    /**
     * 校验密码并解锁系统
     */
    async handleUnlock()
    {
      if (!this.password) {
        this.showError('请输入密码')
        return
      }
      this.loading = true
      this.errorMsg = ''
      try {
        await unlockScreen(this.password)
        const lockPath = this.$store.getters.lockPath // 取锁屏前的路径
        await this.$store.dispatch('lock/unlockScreen')
        this.$router.replace(lockPath)
      } catch (err) {
        const msg = err.message || err.toString()
        this.showError(msg)
        this.password = ''
        this.$refs.passwordInput && this.$refs.passwordInput.focus()
      } finally {
        this.loading = false
      }
    },

    /**
     * 展示错误提示与抖动动画
     * @param {String} msg - 错误提示信息
     */
    showError(msg)
    {
      this.errorMsg = msg
      this.isShaking = true
      setTimeout(() => {
        this.isShaking = false
      }, 600)
    },

    /**
     * 退出登录并跳转登录页
     */
    goLogin()
    {
      this.$store.dispatch('lock/unlockScreen')
      this.$store.dispatch('LogOut').then(() => {
        this.$router.push('/login')
      })
    },

    // 粒子背景
    /**
     * 初始化粒子背景动画
     */
    initParticles()
    {
      const canvas = this.$refs.particleCanvas
      if (!canvas) return
      const ctx = canvas.getContext('2d')
      const resize = () => {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      }
      resize()
      window.addEventListener('resize', resize)
      const count = 80
      for (let i = 0; i < count; i++) {
        this.particles.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          r: Math.random() * 2 + 1,
          dx: (Math.random() - 0.5) * 0.6,
          dy: (Math.random() - 0.5) * 0.6,
          alpha: Math.random() * 0.5 + 0.2,
        })
      }
      const draw = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
        this.particles.forEach((p) => {
          ctx.beginPath()
          ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
          ctx.fillStyle = `rgba(255,255,255,${p.alpha})`
          ctx.fill()
          p.x += p.dx
          p.y += p.dy
          if (p.x < 0 || p.x > canvas.width) p.dx *= -1
          if (p.y < 0 || p.y > canvas.height) p.dy *= -1
        })
        // 连线
        for (let i = 0; i < this.particles.length; i++) {
          for (let j = i + 1; j < this.particles.length; j++) {
            const a = this.particles[i],
              b = this.particles[j]
            const dist = Math.hypot(a.x - b.x, a.y - b.y)
            if (dist < 120) {
              ctx.beginPath()
              ctx.moveTo(a.x, a.y)
              ctx.lineTo(b.x, b.y)
              ctx.strokeStyle = `rgba(255,255,255,${0.15 * (1 - dist / 120)})`
              ctx.lineWidth = 0.5
              ctx.stroke()
            }
          }
        }
        this.animationId = requestAnimationFrame(draw)
      }
      draw()
    },
  },
}
</script>