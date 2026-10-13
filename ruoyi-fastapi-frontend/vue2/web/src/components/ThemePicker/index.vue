<style lang="scss">
.theme-message, .theme-picker-dropdown { z-index: 99999 !important; }
.theme-picker .el-color-picker__trigger { height: 26px !important; width: 26px !important; padding: 2px; }
.theme-picker-dropdown .el-color-dropdown__link-btn { display: none; }
</style>

<template>
  <el-color-picker
    class="theme-picker"
    v-model="theme"
    :predefine="[
      '#409EFF',
      '#1890ff',
      '#304156',
      '#212121',
      '#11a983',
      '#13c2c2',
      '#6959CD',
      '#f5222d',
    ]"
    popper-class="theme-picker-dropdown"/>
</template>

<script>
/**
 * 组件名称：ThemePicker
 * 功能说明：主题色选择组件，支持动态切换主题色
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event change - 主题色变更
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

const ORIGINAL_THEME = '#409EFF' // default color

export default
{
  data()
  {
    return {
      chalk: '', // content of theme-chalk css
      theme: ''
    }
  },

  computed:
  {
    /**
     * 读取默认主题色
     * @returns {String} 默认主题色
     */
    defaultTheme()
    {
      return this.$store.state.settings.theme
    }
  },

  watch:
  {
    defaultTheme:
    {
      /**
       * 同步默认主题色
       * @param {String} val - 新的主题色
       * @param {String} oldVal - 旧的主题色
       */
      handler: function (val, oldVal)
      {
        this.theme = val
      },
      immediate: true
    },

    /**
     * 监听主题色变化并应用
     * @param {String} val - 新的主题色
     */
    async theme(val)
    {
      await this.setTheme(val)
    }
  },

  created()
  {
    if (this.defaultTheme !== ORIGINAL_THEME)
    {
      this.setTheme(this.defaultTheme)
    }
  },

  methods:
  {
    /**
     * 应用指定主题色
     * @param {String} val - 主题色
     */
    async setTheme(val)
    {
      const oldVal = this.chalk ? this.theme : ORIGINAL_THEME
      if (typeof val !== 'string') return
      const themeCluster = this.getThemeCluster(val.replace('#', ''))
      const originalCluster = this.getThemeCluster(oldVal.replace('#', ''))

      const getHandler = (variable, id) =>
      {
        return () =>
        {
          const originalCluster = this.getThemeCluster(ORIGINAL_THEME.replace('#', ''))
          const newStyle = this.updateStyle(this[variable], originalCluster, themeCluster)

          let styleTag = document.getElementById(id)
          if (!styleTag)
          {
            styleTag = document.createElement('style')
            styleTag.setAttribute('id', id)
            document.head.appendChild(styleTag)
          }
          styleTag.innerText = newStyle
        }
      }

      if (!this.chalk)
      {
        const url = `/styles/theme-chalk/index.css`
        await this.getCSSString(url, 'chalk')
      }

      const chalkHandler = getHandler('chalk', 'chalk-style')
      chalkHandler()

      const styles = [].slice.call(document.querySelectorAll('style')).filter((style) =>
      {
        const text = style.innerText
        return new RegExp(oldVal, 'i').test(text) && !/Chalk Variables/.test(text)
      })
      styles.forEach((style) =>
      {
        const { innerText } = style
        if (typeof innerText !== 'string') return
        style.innerText = this.updateStyle(innerText, originalCluster, themeCluster)
      })

      this.$emit('change', val)
    },

    /**
     * 替换样式中的颜色变量
     * @param {String} style - 原始样式
     * @param {Array} oldCluster - 旧颜色集合
     * @param {Array} newCluster - 新颜色集合
     * @returns {String} 替换后的样式
     */
    updateStyle(style, oldCluster, newCluster)
    {
      let newStyle = style
      oldCluster.forEach((color, index) =>
      {
        newStyle = newStyle.replace(new RegExp(color, 'ig'), newCluster[index])
      })
      return newStyle
    },

    /**
     * 请求主题样式文件内容
     * @param {String} url - 样式文件地址
     * @param {String} variable - 存储内容的变量名
     * @returns {Promise} 请求完成的 Promise
     */
    getCSSString(url, variable)
    {
      return new Promise((resolve) =>
      {
        const xhr = new XMLHttpRequest()
        xhr.onreadystatechange = () =>
        {
          if (xhr.readyState === 4 && xhr.status === 200)
          {
            this[variable] = xhr.responseText.replace(/@font-face{[^}]+}/, '')
            resolve()
          }
        }
        xhr.open('GET', url)
        xhr.send()
      })
    },

    /**
     * 生成主题色的色阶集合
     * @param {String} theme - 主题色
     * @returns {Array} 色阶集合
     */
    getThemeCluster(theme)
    {
      const tintColor = (color, tint) =>
      {
        let red = parseInt(color.slice(0, 2), 16)
        let green = parseInt(color.slice(2, 4), 16)
        let blue = parseInt(color.slice(4, 6), 16)

        if (tint === 0)
        {
          // when primary color is in its rgb space
          return [red, green, blue].join(',')
        } else
        {
          red += Math.round(tint * (255 - red))
          green += Math.round(tint * (255 - green))
          blue += Math.round(tint * (255 - blue))

          red = red.toString(16)
          green = green.toString(16)
          blue = blue.toString(16)

          return `#${red}${green}${blue}`
        }
      }

      const shadeColor = (color, shade) =>
      {
        let red = parseInt(color.slice(0, 2), 16)
        let green = parseInt(color.slice(2, 4), 16)
        let blue = parseInt(color.slice(4, 6), 16)

        red = Math.round((1 - shade) * red)
        green = Math.round((1 - shade) * green)
        blue = Math.round((1 - shade) * blue)

        red = red.toString(16)
        green = green.toString(16)
        blue = blue.toString(16)

        return `#${red}${green}${blue}`
      }

      const clusters = [theme]
      for (let i = 0; i <= 9; i++)
      {
        clusters.push(tintColor(theme, Number((i / 10).toFixed(2))))
      }
      clusters.push(shadeColor(theme, 0.1))
      return clusters
    }
  }
}
</script>
