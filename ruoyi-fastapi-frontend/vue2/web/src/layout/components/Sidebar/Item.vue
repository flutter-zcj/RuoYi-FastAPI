<script>
/**
 * 组件名称：Sidebar
 * 功能说明：侧边栏菜单项渲染组件，根据图标与标题生成菜单内容
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} icon  - 菜单图标
 * @prop {String} title - 菜单标题
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
  name: 'MenuItem',
  functional: true,

  props:
  {
    icon: { type: String, default: '' },
    title: { type: String, default: '' }
  },

  /**
   * 渲染菜单项内容
   * @param {Function} h - createElement 函数
   * @param {Object} context - 渲染上下文
   * @returns {Array} 渲染节点集合
   */
  render(h, context)
  {
    const { icon, title } = context.props
    const vnodes = []

    if (icon)
    {
      vnodes.push(<svg-icon icon-class={icon} />)
    }

    if (title)
    {
      if (title.length > 5)
      {
        vnodes.push(
          <span
            slot="title"
            title={title}>
            {title}
          </span>
        )
      }
      else
      {
        vnodes.push(<span slot="title">{title}</span>)
      }
    }
    return vnodes
  }
}
</script>