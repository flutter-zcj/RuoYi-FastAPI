<style lang="scss" scoped>
.pan-item { position: relative; display: inline-block; width: 200px; height: 200px; box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2); border-radius: 50%; cursor: default; }
.pan-info-roles-container { padding: 20px; text-align: center; }
.pan-thumb { position: absolute; overflow: hidden; width: 100%; height: 100%; background-position: center center; background-size: cover; border-radius: 50%; transform-origin: 95% 40%; transition: all 0.3s ease-in-out; }
.pan-info { position: absolute; overflow: hidden; width: inherit; height: inherit; box-shadow: inset 0 0 0 5px rgba(0, 0, 0, 0.05); border-radius: 50%; }
.pan-info h3 { position: relative; height: 85px; margin: 0 60px; padding: 22px 0 0 0; font-family: 'Open Sans', Arial, sans-serif; font-size: 18px; color: #fff; letter-spacing: 2px; text-transform: uppercase; text-shadow: 0 0 1px #fff, 0 1px 2px rgba(0, 0, 0, 0.3); }
.pan-info p { padding: 10px 5px; border-top: 1px solid rgba(255, 255, 255, 0.5); margin: 0 30px; font-style: italic; font-size: 12px; color: #fff; }
.pan-info p a { display: block; width: 80px; height: 80px; background: rgba(255, 255, 255, 0.3); margin: 7px auto 0; padding-top: 24px; font-family: 'Open Sans', Arial, sans-serif; font-size: 9px; font-style: normal; font-weight: 700; color: #333; color: #fff; letter-spacing: 1px; text-transform: uppercase; border-radius: 50%; opacity: 0; transition: transform 0.3s ease-in-out 0.2s, opacity 0.3s ease-in-out 0.2s, background 0.2s linear 0s; transform: translateX(60px) rotate(90deg); }
.pan-info p a:hover { background: rgba(255, 255, 255, 0.5); }
.pan-item:hover .pan-thumb { transform: rotate(-110deg); }
.pan-item:hover .pan-info p a { opacity: 1; transform: translateX(0px) rotate(0deg); }
</style>

<template>
  <div class="pan-item" :style="{ zIndex: zIndex, height: height, width: width }">
    <div class="pan-info">
      <div class="pan-info-roles-container">
        <slot />
      </div>
    </div>
    <div class="pan-thumb" :style="{ backgroundImage: `url(${image})` }"></div>
  </div>
</template>

<script>
/**
 * 组件名称：PanThumb
 * 功能说明：圆形头像缩略图组件，用于展示用户头像与角色信息
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} image  - 头像图片地址
 * @prop {Number} zIndex - 层级
 * @prop {String} width  - 宽度
 * @prop {String} height - 高度
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
  name: 'PanThumb',

  props:
  {
    image: { type: String, required: true },
    zIndex: { type: Number, default: 1 },
    width: { type: String, default: '150px' },
    height: { type: String, default: '150px' }
  }
}
</script>