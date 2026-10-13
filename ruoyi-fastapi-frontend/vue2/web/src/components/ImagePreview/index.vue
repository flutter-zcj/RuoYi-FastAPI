<style lang="scss" scoped>
.el-image { border-radius: 5px; background-color: #ebeef5; box-shadow: 0 0 5px 1px #ccc;
  ::v-deep .el-image__inner { cursor: pointer; transition: all 0.3s;
    &:hover { transform: scale(1.2); }
  }
  ::v-deep .image-slot { display: flex; justify-content: center; align-items: center; width: 100%; height: 100%; color: #909399; font-size: 30px; }
}
</style>

<template>
  <el-image
    :src="`${realSrc}`"
    fit="cover"
    :style="`width:${realWidth};height:${realHeight};`"
    :preview-src-list="realSrcList">
    <div
      class="image-slot"
      slot="error">
      <i class="el-icon-picture-outline"></i>
    </div>
  </el-image>
</template>

<script>
/**
 * 组件名称：ImagePreview
 * 功能说明：图片预览组件，支持多图预览与加载失败占位
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String}           src    - 图片地址
 * @prop {[Number, String]} width  - 图片宽度
 * @prop {[Number, String]} height - 图片高度
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

import { isExternal } from '@/utils/validate'

export default
{
  name: 'ImagePreview',

  props:
  {
    src: { type: String, default: '' },
    width: { type: [Number, String], default: '' },
    height: { type: [Number, String], default: '' }
  },

  computed:
  {
    /**
     * 计算真实图片地址，外部地址直接返回，否则拼接接口基础地址
     * @returns {String} 真实图片地址
     */
    realSrc()
    {
      if (!this.src)
      {
        return
      }
      let real_src = this.src.split(',')[0]
      if (isExternal(real_src))
      {
        return real_src
      }
      return process.env.VUE_APP_BASE_API + real_src
    },

    /**
     * 计算多图预览地址列表
     * @returns {Array} 预览地址列表
     */
    realSrcList()
    {
      if (!this.src)
      {
        return
      }
      let real_src_list = this.src.split(',')
      let srcList = []
      real_src_list.forEach((item) =>
      {
        if (isExternal(item))
        {
          return srcList.push(item)
        }
        return srcList.push(process.env.VUE_APP_BASE_API + item)
      })
      return srcList
    },

    /**
     * 计算图片宽度样式值
     * @returns {String} 宽度值
     */
    realWidth()
    {
      return typeof this.width == 'string' ? this.width : `${this.width}px`
    },

    /**
     * 计算图片高度样式值
     * @returns {String} 高度值
     */
    realHeight()
    {
      return typeof this.height == 'string' ? this.height : `${this.height}px`
    }
  }
}
</script>