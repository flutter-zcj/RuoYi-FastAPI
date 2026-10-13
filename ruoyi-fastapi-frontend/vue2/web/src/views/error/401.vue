<style lang="scss" scoped>
.errPage-container { width: 800px; max-width: 100%; margin: 100px auto;
  .pan-back-btn { background: #008489; border: none !important; color: #fff; }
  .pan-gif { display: block; margin: 0 auto; }
  .pan-img { display: block; width: 100%; margin: 0 auto; }
  .text-jumbo { font-size: 60px; font-weight: 700; color: #484848; }
  .list-unstyled { font-size: 14px;
    li { padding-bottom: 5px; }
    a { color: #008489; text-decoration: none;
      &:hover { text-decoration: underline; }
    }
  }
}
</style>

<template>
  <div class="errPage-container">
    <el-button
      class="pan-back-btn"
      icon="arrow-left"
      @click="back">返回</el-button>
    <el-row>
      <el-col :span="12">
        <h1 class="text-jumbo text-ginormous">401错误!</h1>
        <h2>您没有访问权限！</h2>
        <h6>对不起，您没有访问权限，请不要进行非法操作！您可以返回主页面</h6>
        <ul class="list-unstyled">
          <li class="link-type">
            <router-link to="/"> 回首页 </router-link>
          </li>
        </ul>
      </el-col>
      <el-col :span="12">
        <img
          :src="errGif"
          width="313"
          height="428"
          alt="Girl has dropped her ice cream."/>
      </el-col>
    </el-row>
  </div>
</template>

<script>
/**
 * 组件名称：Page401
 * 功能说明：401 错误页面，提示无访问权限并提供返回入口
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String} errGif - 401 提示图片地址
 *
 * 事件说明：
 * @event back - 返回上一页或跳转首页
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import errGif from '@/assets/401_images/401.gif'

export default
{
  name: 'Page401',

  data()
  {
    return {
      errGif: errGif + '?' + +new Date(),
    }
  },

  methods:
  {
    /**
     * 返回上一页或跳转首页
     */
    back()
    {
      if (this.$route.query.noGoBack) {
        this.$router.push({ path: '/' })
      } else {
        this.$router.go(-1)
      }
    },
  },
}
</script>