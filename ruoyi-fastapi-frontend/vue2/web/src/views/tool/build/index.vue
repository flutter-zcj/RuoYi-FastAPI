<style lang="scss">
.editor-tabs { background: #121315;
  .el-tabs__header { margin: 0; border-bottom-color: #121315;
    .el-tabs__nav { border-color: #121315; }
  }
  .el-tabs__item { height: 32px; line-height: 32px; color: #888a8e; border-left: 1px solid #121315 !important; background: #363636; margin-right: 5px; user-select: none; }
  .el-tabs__item.is-active { background: #1e1e1e; border-bottom-color: #1e1e1e !important; color: #fff; }
  .el-icon-edit { color: #f1fa8c; }
  .el-icon-document { color: #a95812; }
}
.right-scrollbar {
  .el-scrollbar__view { padding: 12px 18px 15px 15px; }
}
.left-scrollbar .el-scrollbar__wrap { box-sizing: border-box; overflow-x: hidden !important; margin-bottom: 0 !important; }
.center-tabs {
  .el-tabs__header { margin-bottom: 0 !important; }
  .el-tabs__item { width: 50%; text-align: center; }
  .el-tabs__nav { width: 100%; }
}
.reg-item { padding: 12px 6px; background: #f8f8f8; position: relative; border-radius: 4px;
  .close-btn { position: absolute; right: -6px; top: -6px; display: block; width: 16px; height: 16px; line-height: 16px; background: rgba(0, 0, 0, 0.2); border-radius: 50%; color: #fff; text-align: center; z-index: 1; cursor: pointer; font-size: 12px;
    &:hover { background: rgba(210, 23, 23, 0.5); }
  }
  & + .reg-item { margin-top: 18px; }
}
.action-bar {
  & .el-button + .el-button { margin-left: 15px; }
  & i { font-size: 20px; vertical-align: middle; position: relative; top: -1px; }
}
.custom-tree-node { width: 100%; font-size: 14px;
  .node-operation { float: right; }
  i[class*='el-icon'] + i[class*='el-icon'] { margin-left: 6px; }
  .el-icon-plus { color: #409eff; }
  .el-icon-delete { color: #157a0c; }
}
.left-scrollbar .el-scrollbar__view { overflow-x: hidden; }
.el-rate { display: inline-block; vertical-align: text-top; }
.el-upload__tip { line-height: 1.2; }
$selectedColor: #f6f7ff;
$lighterBlue: #409eff;
.container { position: relative; width: 100%; height: 100%; }
.components-list { padding: 8px; box-sizing: border-box; height: 100%;
  .components-item { display: inline-block; width: 48%; margin: 1%; transition: transform 0ms !important; }
}
.components-draggable { padding-bottom: 20px; }
.components-title { font-size: 14px; color: #222; margin: 6px 2px;
  .svg-icon { color: #666; font-size: 18px; }
}
.components-body { padding: 8px 10px; background: $selectedColor; font-size: 12px; cursor: move; border: 1px dashed $selectedColor; border-radius: 3px;
  .svg-icon { color: #777; font-size: 15px; }
  &:hover { border: 1px dashed #787be8; color: #787be8;
    .svg-icon { color: #787be8; }
  }
}
.left-board { width: 260px; position: absolute; left: 0; top: 0; height: 100vh; }
.left-scrollbar { height: calc(100vh - 42px); overflow: hidden; }
.center-scrollbar { height: calc(100vh - 42px); overflow: hidden; border-left: 1px solid #f1e8e8; border-right: 1px solid #f1e8e8; box-sizing: border-box; }
.center-board { height: 100vh; width: auto; margin: 0 350px 0 260px; box-sizing: border-box; }
.empty-info { position: absolute; top: 46%; left: 0; right: 0; text-align: center; font-size: 18px; color: #ccb1ea; letter-spacing: 4px; }
.action-bar { position: relative; height: 42px; text-align: right; padding: 0 15px; box-sizing: border-box; border: 1px solid #f1e8e8; border-top: none; border-left: none;
  .delete-btn { color: #f56c6c; }
}
.logo-wrapper { position: relative; height: 42px; background: #fff; border-bottom: 1px solid #f1e8e8; box-sizing: border-box; }
.logo { position: absolute; left: 12px; top: 6px; line-height: 30px; color: #00afff; font-weight: 600; font-size: 17px; white-space: nowrap;
  > img { width: 30px; height: 30px; vertical-align: top; }
  .github { display: inline-block; vertical-align: sub; margin-left: 15px;
    > img { height: 22px; }
  }
}
.center-board-row { padding: 12px 12px 15px 12px; box-sizing: border-box;
  & > .el-form { height: calc(100vh - 69px); }
}
.drawing-board { height: 100%; position: relative;
  .components-body { padding: 0; margin: 0; font-size: 0; }
  .sortable-ghost { position: relative; display: block; overflow: hidden;
    &::before { content: ' '; position: absolute; left: 0; right: 0; top: 0; height: 3px; background: rgb(89, 89, 223); z-index: 2; }
  }
  .components-item.sortable-ghost { width: 100%; height: 60px; background-color: $selectedColor; }
  .active-from-item {
    & > .el-form-item { background: $selectedColor; border-radius: 6px; }
    & > .drawing-item-copy, & > .drawing-item-delete { display: initial; }
    & > .component-name { color: $lighterBlue; }
  }
  .el-form-item { margin-bottom: 15px; }
}
.drawing-item { position: relative; cursor: move;
  &.unfocus-bordered:not(.activeFromItem) > div:first-child { border: 1px dashed #ccc; }
  .el-form-item { padding: 12px 10px; }
}
.drawing-row-item { position: relative; cursor: move; box-sizing: border-box; border: 1px dashed #ccc; border-radius: 3px; padding: 0 2px; margin-bottom: 15px;
  .drawing-row-item { margin-bottom: 2px; }
  .el-col { margin-top: 22px; }
  .el-form-item { margin-bottom: 0; }
  .drag-wrapper { min-height: 80px; }
  &.active-from-item { border: 1px dashed $lighterBlue; }
  .component-name { position: absolute; top: 0; left: 0; font-size: 12px; color: #bbb; display: inline-block; padding: 0 6px; }
}
.drawing-item, .drawing-row-item {
  &:hover {
    & > .el-form-item { background: $selectedColor; border-radius: 6px; }
    & > .drawing-item-copy, & > .drawing-item-delete { display: initial; }
  }
  & > .drawing-item-copy, & > .drawing-item-delete { display: none; position: absolute; top: -10px; width: 22px; height: 22px; line-height: 22px; text-align: center; border-radius: 50%; font-size: 12px; border: 1px solid; cursor: pointer; z-index: 1; }
  & > .drawing-item-copy { right: 56px; border-color: $lighterBlue; color: $lighterBlue; background: #fff;
    &:hover { background: $lighterBlue; color: #fff; }
  }
  & > .drawing-item-delete { right: 24px; border-color: #f56c6c; color: #f56c6c; background: #fff;
    &:hover { background: #f56c6c; color: #fff; }
  }
}
</style>

<template>
  <div class="container">
    <div class="left-board">
      <div class="logo-wrapper">
        <div class="logo">
          <img :src="logo" alt="logo" />
          Form Generator
        </div>
      </div>
      <el-scrollbar class="left-scrollbar">
        <div class="components-list">
          <div class="components-title"><svg-icon icon-class="component" />输入型组件</div>
          <draggable
            class="components-draggable"
            :list="inputComponents"
            :group="{ name: 'componentsGroup', pull: 'clone', put: false }"
            :clone="cloneComponent"
            draggable=".components-item"
            :sort="false"
            @end="onEnd">
            <div
              class="components-item"
              v-for="(element, index) in inputComponents"
              :key="index"
              @click="addComponent(element)">
              <div class="components-body">
                <svg-icon :icon-class="element.tagIcon" />
                {{ element.label }}
              </div>
            </div>
          </draggable>
          <div class="components-title"><svg-icon icon-class="component" />选择型组件</div>
          <draggable
            class="components-draggable"
            :list="selectComponents"
            :group="{ name: 'componentsGroup', pull: 'clone', put: false }"
            :clone="cloneComponent"
            draggable=".components-item"
            :sort="false"
            @end="onEnd">
            <div
              class="components-item"
              v-for="(element, index) in selectComponents"
              :key="index"
              @click="addComponent(element)">
              <div class="components-body">
                <svg-icon :icon-class="element.tagIcon" />
                {{ element.label }}
              </div>
            </div>
          </draggable>
          <div class="components-title"><svg-icon icon-class="component" /> 布局型组件</div>
          <draggable
            class="components-draggable"
            :list="layoutComponents"
            :group="{ name: 'componentsGroup', pull: 'clone', put: false }"
            :clone="cloneComponent"
            draggable=".components-item"
            :sort="false"
            @end="onEnd">
            <div
              class="components-item"
              v-for="(element, index) in layoutComponents"
              :key="index"
              @click="addComponent(element)">
              <div class="components-body">
                <svg-icon :icon-class="element.tagIcon" />
                {{ element.label }}
              </div>
            </div>
          </draggable>
        </div>
      </el-scrollbar>
    </div>

    <div class="center-board">
      <div class="action-bar">
        <el-button
          icon="el-icon-download"
          type="text"
          @click="download">
          导出vue文件
        </el-button>
        <el-button
          class="copy-btn-main"
          icon="el-icon-document-copy"
          type="text"
          @click="copy">
          复制代码
        </el-button>
        <el-button
          class="delete-btn"
          icon="el-icon-delete"
          type="text"
          @click="empty">
          清空
        </el-button>
      </div>
      <el-scrollbar class="center-scrollbar">
        <el-row class="center-board-row" :gutter="formConf.gutter">
          <el-form
            :size="formConf.size"
            :label-position="formConf.labelPosition"
            :disabled="formConf.disabled"
            :label-width="formConf.labelWidth + 'px'">
            <draggable
              class="drawing-board"
              :list="drawingList"
              :animation="340"
              group="componentsGroup">
              <draggable-item
                v-for="(element, index) in drawingList"
                :key="element.renderKey"
                :drawing-list="drawingList"
                :element="element"
                :index="index"
                :active-id="activeId"
                :form-conf="formConf"
                @activeItem="activeFormItem"
                @copyItem="drawingItemCopy"
                @deleteItem="drawingItemDelete"/>
            </draggable>
            <div class="empty-info" v-show="!drawingList.length">
              从左侧拖入或点选组件进行表单设计
            </div>
          </el-form>
        </el-row>
      </el-scrollbar>
    </div>

    <right-panel
      :active-data="activeData"
      :form-conf="formConf"
      :show-field="!!drawingList.length"
      @tag-change="tagChange"/>

    <code-type-dialog
      :visible.sync="dialogVisible"
      title="选择生成类型"
      :show-file-name="showFileName"
      @confirm="generate"/>
    <input id="copyNode" type="hidden" />
  </div>
</template>

<script>
/**
 * 组件名称：build
 * 功能说明：表单生成器页面，支持拖拽设计表单并生成对应代码
 * 研发时间：2024/01/25
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {String}  logo              - 项目 logo 图片地址
 * @data {Number}  idGlobal          - 组件自增 ID
 * @data {Object}  formConf          - 表单配置
 * @data {Array}   inputComponents   - 输入型组件列表
 * @data {Array}   selectComponents  - 选择型组件列表
 * @data {Array}   layoutComponents  - 布局型组件列表
 * @data {Number}  labelWidth        - 标签宽度
 * @data {Array}   drawingList       - 设计区组件列表
 * @data {Object}  drawingData       - 设计区数据
 * @data {Number}  activeId          - 当前激活组件 ID
 * @data {Boolean} drawerVisible     - 是否显示运行抽屉
 * @data {Object}  formData          - 组装后的表单数据
 * @data {Boolean} dialogVisible     - 是否显示生成类型弹窗
 * @data {Object}  generateConf      - 生成配置
 * @data {Boolean} showFileName      - 是否显示文件名
 * @data {Object}  activeData        - 当前激活组件数据
 *
 * 事件说明：
 * @event activeFormItem     - 激活当前选中组件
 * @event onEnd              - 拖拽结束回调
 * @event addComponent       - 点击添加组件
 * @event cloneComponent     - 克隆组件
 * @event AssembleFormData   - 组装表单数据
 * @event generate           - 根据操作类型生成
 * @event execRun            - 运行表单
 * @event execDownload       - 下载生成的文件
 * @event execCopy           - 复制生成的代码
 * @event empty              - 清空设计区
 * @event drawingItemCopy    - 复制设计区组件
 * @event createIdAndKey     - 重新生成组件 ID 与 renderKey
 * @event drawingItemDelete  - 删除设计区组件
 * @event generateCode       - 生成代码字符串
 * @event download           - 打开下载弹窗
 * @event run                - 打开运行弹窗
 * @event copy               - 打开复制弹窗
 * @event tagChange          - 组件标签切换
 * @event updateDrawingList  - 更新设计区组件
 *
 * 插槽说明：
 */

/**
 * 2024/01/25     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import draggable from 'vuedraggable'
import beautifier from 'js-beautify'
import ClipboardJS from 'clipboard'
import render from '@/utils/generator/render'
import RightPanel from './RightPanel'
import {
  inputComponents,
  selectComponents,
  layoutComponents,
  formConf,
} from '@/utils/generator/config'
import { beautifierConf, titleCase } from '@/utils/index'
import { makeUpHtml, vueTemplate, vueScript, cssStyle } from '@/utils/generator/html'
import { makeUpJs } from '@/utils/generator/js'
import { makeUpCss } from '@/utils/generator/css'
import {
  drawingDefaultValue,
  initDrawingDefaultValue,
  cleanDrawingDefaultValue,
} from '@/utils/generator/drawingDefault'
import logo from '@/assets/logo/logo.png'
import CodeTypeDialog from './CodeTypeDialog'
import DraggableItem from './DraggableItem'

let oldActiveId
let tempActiveData
let clipboard = null

export default
{
  components:
  {
    draggable,
    render,
    RightPanel,
    CodeTypeDialog,
    DraggableItem,
  },

  data()
  {
    return {
      logo,
      idGlobal: 100,
      formConf,
      inputComponents,
      selectComponents,
      layoutComponents,
      labelWidth: 100,
      drawingList: drawingDefaultValue,
      drawingData: {},
      activeId: drawingDefaultValue[0].formId,
      drawerVisible: false,
      formData: {},
      dialogVisible: false,
      generateConf: null,
      showFileName: false,
      activeData: drawingDefaultValue[0],
    }
  },

  watch:
  {
    'activeData.label': function (val, oldVal)
    {
      if (
        this.activeData.placeholder === undefined ||
        !this.activeData.tag ||
        oldActiveId !== this.activeId
      )
      {
        return
      }
      this.activeData.placeholder = this.activeData.placeholder.replace(oldVal, '') + val
    },

    activeId:
    {
      handler(val)
      {
        oldActiveId = val
      },
      immediate: true,
    },
  },

  beforeCreate()
  {
    initDrawingDefaultValue()
  },

  created()
  {
    // 防止 firefox 下 拖拽 会新打卡一个选项卡
    document.body.ondrop = (event) =>
    {
      event.preventDefault()
      event.stopPropagation()
    }
  },

  mounted()
  {
    clipboard = new ClipboardJS('#copyNode', {
      text: (trigger) =>
      {
        const codeStr = this.generateCode()
        this.$notify({
          title: '成功',
          message: '代码已复制到剪切板，可粘贴。',
          type: 'success',
        })
        return codeStr
      },
    })
    clipboard.on('error', (e) =>
    {
      this.$message.error('代码复制失败')
    })
  },

  beforeDestroy()
  {
    clipboard.destroy()
  },

  methods:
  {
    /**
     * 激活当前选中组件
     * @param {Object} element - 组件数据
     */
    activeFormItem(element)
    {
      this.activeData = element
      this.activeId = element.formId
    },

    /**
     * 拖拽结束回调
     * @param {Object} obj - 拖拽事件对象
     * @param {*} a - 拖拽事件附加参数
     */
    onEnd(obj, a)
    {
      if (obj.from !== obj.to)
      {
        this.activeData = tempActiveData
        this.activeId = this.idGlobal
      }
    },

    /**
     * 点击添加组件
     * @param {Object} item - 组件配置
     */
    addComponent(item)
    {
      const clone = this.cloneComponent(item)
      this.drawingList.push(clone)
      this.activeFormItem(clone)
    },

    /**
     * 克隆组件
     * @param {Object} origin - 原始组件配置
     * @returns {Object} 克隆后的组件配置
     */
    cloneComponent(origin)
    {
      const clone = JSON.parse(JSON.stringify(origin))
      clone.formId = ++this.idGlobal
      clone.span = formConf.span
      clone.renderKey = +new Date() // 改变renderKey后可以实现强制更新组件
      if (!clone.layout) clone.layout = 'colFormItem'
      if (clone.layout === 'colFormItem')
      {
        clone.vModel = `field${this.idGlobal}`
        clone.placeholder !== undefined && (clone.placeholder += clone.label)
        tempActiveData = clone
      }
      else if (clone.layout === 'rowFormItem')
      {
        delete clone.label
        clone.componentName = `row${this.idGlobal}`
        clone.gutter = this.formConf.gutter
        tempActiveData = clone
      }
      return tempActiveData
    },

    /**
     * 组装表单数据
     */
    AssembleFormData()
    {
      this.formData = {
        fields: JSON.parse(JSON.stringify(this.drawingList)),
        ...this.formConf,
      }
    },

    /**
     * 根据操作类型生成
     * @param {Object} data - 生成配置数据
     */
    generate(data)
    {
      const func = this[`exec${titleCase(this.operationType)}`]
      this.generateConf = data
      func && func(data)
    },

    /**
     * 运行表单
     * @param {Object} data - 生成配置数据
     */
    execRun(data)
    {
      this.AssembleFormData()
      this.drawerVisible = true
    },

    /**
     * 下载生成的文件
     * @param {Object} data - 生成配置数据
     */
    execDownload(data)
    {
      const codeStr = this.generateCode()
      const blob = new Blob([codeStr], { type: 'text/plain;charset=utf-8' })
      this.$download.saveAs(blob, data.fileName)
    },

    /**
     * 复制生成的代码
     * @param {Object} data - 生成配置数据
     */
    execCopy(data)
    {
      document.getElementById('copyNode').click()
    },

    /**
     * 清空设计区
     */
    empty()
    {
      this.$confirm('确定要清空所有组件吗？', '提示', { type: 'warning' }).then(() =>
      {
        this.drawingList = []
        cleanDrawingDefaultValue()
      })
    },

    /**
     * 复制设计区组件
     * @param {Object} item - 组件数据
     * @param {Array} parent - 父级组件数组
     */
    drawingItemCopy(item, parent)
    {
      let clone = JSON.parse(JSON.stringify(item))
      clone = this.createIdAndKey(clone)
      parent.push(clone)
      this.activeFormItem(clone)
    },

    /**
     * 重新生成组件 ID 与 renderKey
     * @param {Object} item - 组件数据
     * @returns {Object} 更新后的组件数据
     */
    createIdAndKey(item)
    {
      item.formId = ++this.idGlobal
      item.renderKey = +new Date()
      if (item.layout === 'colFormItem')
      {
        item.vModel = `field${this.idGlobal}`
      }
      else if (item.layout === 'rowFormItem')
      {
        item.componentName = `row${this.idGlobal}`
      }
      if (Array.isArray(item.children))
      {
        item.children = item.children.map((childItem) => this.createIdAndKey(childItem))
      }
      return item
    },

    /**
     * 删除设计区组件
     * @param {Number} index - 组件所在索引
     * @param {Array} parent - 父级组件数组
     */
    drawingItemDelete(index, parent)
    {
      parent.splice(index, 1)
      this.$nextTick(() =>
      {
        const len = this.drawingList.length
        if (len)
        {
          this.activeFormItem(this.drawingList[len - 1])
        }
      })
    },

    /**
     * 生成代码字符串
     * @returns {String} 生成后的代码字符串
     */
    generateCode()
    {
      const { type } = this.generateConf
      this.AssembleFormData()
      const script = vueScript(makeUpJs(this.formData, type))
      const html = vueTemplate(makeUpHtml(this.formData, type))
      const css = cssStyle(makeUpCss(this.formData))
      return beautifier.html(html + script + css, beautifierConf.html)
    },

    /**
     * 打开下载弹窗
     */
    download()
    {
      this.dialogVisible = true
      this.showFileName = true
      this.operationType = 'download'
    },

    /**
     * 打开运行弹窗
     */
    run()
    {
      this.dialogVisible = true
      this.showFileName = false
      this.operationType = 'run'
    },

    /**
     * 打开复制弹窗
     */
    copy()
    {
      this.dialogVisible = true
      this.showFileName = false
      this.operationType = 'copy'
    },

    /**
     * 组件标签切换
     * @param {Object} newTag - 新的组件标签
     */
    tagChange(newTag)
    {
      newTag = this.cloneComponent(newTag)
      newTag.vModel = this.activeData.vModel
      newTag.formId = this.activeId
      newTag.span = this.activeData.span
      delete this.activeData.tag
      delete this.activeData.tagIcon
      delete this.activeData.document
      Object.keys(newTag).forEach((key) =>
      {
        if (
          this.activeData[key] !== undefined &&
          typeof this.activeData[key] === typeof newTag[key]
        )
        {
          newTag[key] = this.activeData[key]
        }
      })
      this.activeData = newTag
      this.updateDrawingList(newTag, this.drawingList)
    },

    /**
     * 更新设计区组件
     * @param {Object} newTag - 新的组件数据
     * @param {Array} list - 组件列表
     */
    updateDrawingList(newTag, list)
    {
      const index = list.findIndex((item) => item.formId === this.activeId)
      if (index > -1)
      {
        list.splice(index, 1, newTag)
      }
      else
      {
        list.forEach((item) =>
        {
          if (Array.isArray(item.children)) this.updateDrawingList(newTag, item.children)
        })
      }
    },
  },
}
</script>