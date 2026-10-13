<style lang="scss" scoped>
.wscn-http404-container { position: absolute; top: 40%; left: 50%; transform: translate(-50%, -50%); }
.wscn-http404 { position: relative; width: 1200px; padding: 0 50px; overflow: hidden;
  .pic-404 { position: relative; float: left; width: 600px; overflow: hidden;
    &__parent { width: 100%; }
    &__child { position: absolute;
      &.left { top: 17px; left: 220px; width: 80px; opacity: 0; animation-name: cloudLeft; animation-duration: 2s; animation-timing-function: linear; animation-fill-mode: forwards; animation-delay: 1s; }
      &.mid { top: 10px; left: 420px; width: 46px; opacity: 0; animation-name: cloudMid; animation-duration: 2s; animation-timing-function: linear; animation-fill-mode: forwards; animation-delay: 1.2s; }
      &.right { top: 100px; left: 500px; width: 62px; opacity: 0; animation-name: cloudRight; animation-duration: 2s; animation-timing-function: linear; animation-fill-mode: forwards; animation-delay: 1s; }
      @keyframes cloudLeft {
        0% { top: 17px; left: 220px; opacity: 0; }
        20% { top: 33px; left: 188px; opacity: 1; }
        80% { top: 81px; left: 92px; opacity: 1; }
        100% { top: 97px; left: 60px; opacity: 0; }
      }
      @keyframes cloudMid {
        0% { top: 10px; left: 420px; opacity: 0; }
        20% { top: 40px; left: 360px; opacity: 1; }
        70% { top: 130px; left: 180px; opacity: 1; }
        100% { top: 160px; left: 120px; opacity: 0; }
      }
      @keyframes cloudRight {
        0% { top: 100px; left: 500px; opacity: 0; }
        20% { top: 120px; left: 460px; opacity: 1; }
        80% { top: 180px; left: 340px; opacity: 1; }
        100% { top: 200px; left: 300px; opacity: 0; }
      }
    }
  }
  .bullshit { position: relative; float: left; width: 300px; padding: 30px 0; overflow: hidden;
    &__oops { margin-bottom: 20px; font-size: 32px; font-weight: bold; color: #1482f0; line-height: 40px; opacity: 0; animation-name: slideUp; animation-duration: 0.5s; animation-fill-mode: forwards; }
    &__headline { margin-bottom: 10px; font-size: 20px; font-weight: bold; color: #222; line-height: 24px; opacity: 0; animation-name: slideUp; animation-duration: 0.5s; animation-delay: 0.1s; animation-fill-mode: forwards; }
    &__info { margin-bottom: 30px; font-size: 13px; color: grey; line-height: 21px; opacity: 0; animation-name: slideUp; animation-duration: 0.5s; animation-delay: 0.2s; animation-fill-mode: forwards; }
    &__return-home { display: block; float: left; width: 110px; height: 36px; border-radius: 100px; background: #1482f0; color: #ffffff; font-size: 14px; text-align: center; line-height: 36px; opacity: 0; cursor: pointer; animation-name: slideUp; animation-duration: 0.5s; animation-delay: 0.3s; animation-fill-mode: forwards; }
    @keyframes slideUp {
      0% { transform: translateY(60px); opacity: 0; }
      100% { transform: translateY(0); opacity: 1; }
    }
  }
}
</style>

<template>
  <div class="wscn-http404-container">
    <div class="wscn-http404">
      <div class="pic-404">
        <img
          class="pic-404__parent"
          src="@/assets/404_images/404.png"
          alt="404"/>
        <img
          class="pic-404__child left"
          src="@/assets/404_images/404_cloud.png"
          alt="404"/>
        <img
          class="pic-404__child mid"
          src="@/assets/404_images/404_cloud.png"
          alt="404"/>
        <img
          class="pic-404__child right"
          src="@/assets/404_images/404_cloud.png"
          alt="404"/>
      </div>
      <div class="bullshit">
        <div class="bullshit__oops">404错误!</div>
        <div class="bullshit__headline">
          {{ message }}
        </div>
        <div class="bullshit__info">
          对不起，您正在寻找的页面不存在。尝试检查URL的错误，然后按浏览器上的刷新按钮或尝试在我们的应用程序中找到其他内容。
        </div>
        <router-link class="bullshit__return-home" to="/">返回首页</router-link>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 组件名称：Page404
 * 功能说明：404 错误页面，提示页面不存在并提供返回首页入口
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'Page404',

  computed:
  {
    /**
     * 页面提示信息
     * @returns {String} 页面提示信息
     */
    message()
    {
      return '找不到网页！'
    },
  },
}
</script>