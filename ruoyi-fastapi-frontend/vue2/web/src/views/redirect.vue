<style lang="scss" scoped></style>

<script>
/**
 * 组件名称：Redirect
 * 功能说明：路由重定向组件，根据路由参数跳转到目标路径
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
  /**
   * 组件创建时根据路由参数执行重定向
   */
  created()
  {
    const { params, query } = this.$route
    const { path } = params
    this.$router.replace({ path: '/' + path, query })
  },

  /**
   * 渲染空节点，避免控制台警告信息
   * @param {Function} h - 渲染函数
   * @returns {VNode} 空虚拟节点
   */
  render: function (h)
  {
    return h() // avoid warning message
  },
}
</script>