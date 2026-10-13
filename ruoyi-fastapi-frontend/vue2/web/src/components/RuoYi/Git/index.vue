<template>
  <div>
    <svg-icon icon-class="github" @click="goto"/>
  </div>
</template>

<script>
/**
 * 组件名称：Git
 * 功能说明：项目源码地址跳转组件，点击图标打开源码仓库
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'RuoYiGit',

  data()
  {
    return {
      url: 'https://gitee.com/insistence2022/RuoYi-Vue-FastAPI'
    }
  },

  methods:
  {
    /**
     * 打开项目源码地址
     */
    goto()
    {
      window.open(this.url)
    }
  }
}
</script>
