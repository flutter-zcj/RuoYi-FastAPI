<style lang="scss" scoped></style>

<script>
/**
 * 组件名称：DraggableItem
 * 功能说明：表单生成器画布中的可拖拽布局项渲染组件
 * 研发时间：2024/12/11
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Object} element     - 当前组件配置对象
 * @prop {Number} index       - 当前组件索引
 * @prop {Array}  drawingList - 画布组件列表
 * @prop {Number} activeId    - 当前选中组件 id
 * @prop {Object} formConf    - 表单配置对象
 *
 * 变量说明：
 *
 * 事件说明：
 * @event render - 根据元素 layout 渲染对应的布局结构
 *
 * 插槽说明：
 */

/**
 * 2024/12/11     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import draggable from 'vuedraggable'
import render from '@/utils/generator/render'

const components = {
  /**
   * 渲染组件项的复制与删除按钮
   * @param {Function} h - createElement 函数
   * @param {Object} element - 当前组件配置对象
   * @param {Number} index - 当前组件索引
   * @param {Array} parent - 父级组件列表
   * @returns {Array} 复制与删除按钮的虚拟节点数组
   */
  itemBtns(h, element, index, parent)
  {
    const { copyItem, deleteItem } = this.$listeners
    return [
      <span
        class="drawing-item-copy"
        title="复制"
        onClick={(event) =>
        {
          copyItem(element, parent)
          event.stopPropagation()
        }}
      >
        <i class="el-icon-copy-document" />
      </span>,
      <span
        class="drawing-item-delete"
        title="删除"
        onClick={(event) =>
        {
          deleteItem(index, parent)
          event.stopPropagation()
        }}
      >
        <i class="el-icon-delete" />
      </span>,
    ]
  },
}
const layouts = {
  /**
   * 渲染列表单项布局
   * @param {Function} h - createElement 函数
   * @param {Object} element - 当前组件配置对象
   * @param {Number} index - 当前组件索引
   * @param {Array} parent - 父级组件列表
   * @returns {VNode} 渲染的虚拟节点
   */
  colFormItem(h, element, index, parent)
  {
    const { activeItem } = this.$listeners
    let className =
      this.activeId === element.formId ? 'drawing-item active-from-item' : 'drawing-item'
    if (this.formConf.unFocusedComponentBorder) className += ' unfocus-bordered'
    return (
      <el-col
        span={element.span}
        class={className}
        nativeOnClick={(event) =>
        {
          activeItem(element)
          event.stopPropagation()
        }}
      >
        <el-form-item
          label-width={element.labelWidth ? `${element.labelWidth}px` : null}
          label={element.label}
          required={element.required}
        >
          <render
            key={element.renderKey}
            conf={element}
            onInput={(event) =>
            {
              this.$set(element, 'defaultValue', event)
            }}
          />
        </el-form-item>
        {components.itemBtns.apply(this, arguments)}
      </el-col>
    )
  },

  /**
   * 渲染行表单项布局
   * @param {Function} h - createElement 函数
   * @param {Object} element - 当前组件配置对象
   * @param {Number} index - 当前组件索引
   * @param {Array} parent - 父级组件列表
   * @returns {VNode} 渲染的虚拟节点
   */
  rowFormItem(h, element, index, parent)
  {
    const { activeItem } = this.$listeners
    const className =
      this.activeId === element.formId ? 'drawing-row-item active-from-item' : 'drawing-row-item'
    let child = renderChildren.apply(this, arguments)
    if (element.type === 'flex')
    {
      child = (
        <el-row
          type={element.type}
          justify={element.justify}
          align={element.align}
        >
          {child}
        </el-row>
      )
    }
    return (
      <el-col span={element.span}>
        <el-row
          gutter={element.gutter}
          class={className}
          nativeOnClick={(event) =>
          {
            activeItem(element)
            event.stopPropagation()
          }}
        >
          <span class="component-name">{element.componentName}</span>
          <draggable
            list={element.children}
            animation={340}
            group="componentsGroup"
            class="drag-wrapper"
          >
            {child}
          </draggable>
          {components.itemBtns.apply(this, arguments)}
        </el-row>
      </el-col>
    )
  },
}

/**
 * 递归渲染子级组件
 * @param {Function} h - createElement 函数
 * @param {Object} element - 当前组件配置对象
 * @param {Number} index - 当前组件索引
 * @param {Array} parent - 父级组件列表
 * @returns {Array} 子级组件虚拟节点数组
 */
function renderChildren(h, element, index, parent)
{
  if (!Array.isArray(element.children)) return null
  return element.children.map((el, i) =>
  {
    const layout = layouts[el.layout]
    if (layout)
    {
      return layout.call(this, h, el, i, element.children)
    }
    return layoutIsNotFound()
  })
}

/**
 * 未匹配到 layout 时抛出异常
 * @returns {VNode} 无返回值，直接抛出异常
 */
function layoutIsNotFound()
{
  throw new Error(`没有与${this.element.layout}匹配的layout`)
}

export default
{
  components: {
    render,
    draggable,
  },

  props: ['element', 'index', 'drawingList', 'activeId', 'formConf'],

  /**
   * 渲染函数，根据元素 layout 调用对应布局渲染
   * @param {Function} h - createElement 函数
   * @returns {VNode} 渲染的虚拟节点
   */
  render(h)
  {
    const layout = layouts[this.element.layout]

    if (layout)
    {
      return layout.call(this, h, this.element, this.index, this.drawingList)
    }
    return layoutIsNotFound()
  },
}
</script>