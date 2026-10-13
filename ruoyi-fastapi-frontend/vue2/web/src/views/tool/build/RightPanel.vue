<style lang="scss" scoped>
.right-board { position: absolute; right: 0; top: 0; width: 350px; padding-top: 3px;
  .field-box { position: relative; overflow: hidden; height: calc(100vh - 42px); box-sizing: border-box; }
  .el-scrollbar { height: 100%; }
}
.select-item { display: flex; border: 1px dashed #fff; box-sizing: border-box;
  & .close-btn { color: #f56c6c; cursor: pointer; }
  & .el-input + .el-input { margin-left: 4px; }
}
.select-item + .select-item { margin-top: 4px; }
.select-item.sortable-chosen { border: 1px dashed #409eff; }
.select-line-icon { padding: 0 4px; font-size: 22px; color: #777; line-height: 32px; }
.option-drag { cursor: move; }
.time-range {
  .el-date-editor { width: 227px; }
  ::v-deep .el-icon-time { display: none; }
}
.document-link { position: absolute; display: block; left: 0; top: 0; z-index: 1; width: 26px; height: 26px; background: #409eff; color: #fff; font-size: 18px; text-align: center; line-height: 26px; cursor: pointer; border-radius: 0 0 6px 0; }
.node-label { font-size: 14px; }
.node-icon { color: #bebfc3; }
</style>

<template>
  <div class="right-board">
    <el-tabs class="center-tabs" v-model="currentTab">
      <el-tab-pane label="组件属性" name="field" />
      <el-tab-pane label="表单属性" name="form" />
    </el-tabs>
    <div class="field-box">
      <a
        class="document-link"
        target="_blank"
        :href="documentLink"
        title="查看组件文档">
        <i class="el-icon-link" />
      </a>
      <el-scrollbar class="right-scrollbar">
        <!-- 组件属性 -->
        <el-form
          v-show="currentTab === 'field' && showField"
          size="small"
          label-width="90px">
          <el-form-item v-if="activeData.changeTag" label="组件类型">
            <el-select
              v-model="activeData.tagIcon"
              placeholder="请选择组件类型"
              :style="{ width: '100%' }"
              @change="tagChange">
              <el-option-group
                v-for="group in tagList"
                :key="group.label"
                :label="group.label">
                <el-option
                  v-for="item in group.options"
                  :key="item.label"
                  :label="item.label"
                  :value="item.tagIcon">
                  <svg-icon class="node-icon" :icon-class="item.tagIcon" />
                  <span> {{ item.label }}</span>
                </el-option>
              </el-option-group>
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.vModel !== undefined" label="字段名">
            <el-input v-model="activeData.vModel" placeholder="请输入字段名（v-model）" />
          </el-form-item>
          <el-form-item v-if="activeData.componentName !== undefined" label="组件名">
            {{ activeData.componentName }}
          </el-form-item>
          <el-form-item v-if="activeData.label !== undefined" label="标题">
            <el-input v-model="activeData.label" placeholder="请输入标题" />
          </el-form-item>
          <el-form-item v-if="activeData.placeholder !== undefined" label="占位提示">
            <el-input v-model="activeData.placeholder" placeholder="请输入占位提示" />
          </el-form-item>
          <el-form-item v-if="activeData['start-placeholder'] !== undefined" label="开始占位">
            <el-input v-model="activeData['start-placeholder']" placeholder="请输入占位提示" />
          </el-form-item>
          <el-form-item v-if="activeData['end-placeholder'] !== undefined" label="结束占位">
            <el-input v-model="activeData['end-placeholder']" placeholder="请输入占位提示" />
          </el-form-item>
          <el-form-item v-if="activeData.span !== undefined" label="表单栅格">
            <el-slider
              v-model="activeData.span"
              :max="24"
              :min="1"
              :marks="{ 12: '' }"
              @change="spanChange"/>
          </el-form-item>
          <el-form-item v-if="activeData.layout === 'rowFormItem'" label="栅格间隔">
            <el-input-number
              v-model="activeData.gutter"
              :min="0"
              placeholder="栅格间隔"/>
          </el-form-item>
          <el-form-item v-if="activeData.layout === 'rowFormItem'" label="布局模式">
            <el-radio-group v-model="activeData.type">
              <el-radio-button label="default" />
              <el-radio-button label="flex" />
            </el-radio-group>
          </el-form-item>
          <el-form-item
            v-if="activeData.justify !== undefined && activeData.type === 'flex'"
            label="水平排列">
            <el-select
              v-model="activeData.justify"
              placeholder="请选择水平排列"
              :style="{ width: '100%' }">
              <el-option
                v-for="(item, index) in justifyOptions"
                :key="index"
                :label="item.label"
                :value="item.value"/>
            </el-select>
          </el-form-item>
          <el-form-item
            v-if="activeData.align !== undefined && activeData.type === 'flex'"
            label="垂直排列">
            <el-radio-group v-model="activeData.align">
              <el-radio-button label="top" />
              <el-radio-button label="middle" />
              <el-radio-button label="bottom" />
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData.labelWidth !== undefined" label="标签宽度">
            <el-input
              v-model.number="activeData.labelWidth"
              type="number"
              placeholder="请输入标签宽度"/>
          </el-form-item>
          <el-form-item
            v-if="activeData.style && activeData.style.width !== undefined"
            label="组件宽度">
            <el-input
              v-model="activeData.style.width"
              placeholder="请输入组件宽度"
              clearable/>
          </el-form-item>
          <el-form-item v-if="activeData.vModel !== undefined" label="默认值">
            <el-input
              :value="setDefaultValue(activeData.defaultValue)"
              placeholder="请输入默认值"
              @input="onDefaultValueInput"/>
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-checkbox-group'" label="至少应选">
            <el-input-number
              :value="activeData.min"
              :min="0"
              placeholder="至少应选"
              @input="$set(activeData, 'min', $event ? $event : undefined)"/>
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-checkbox-group'" label="最多可选">
            <el-input-number
              :value="activeData.max"
              :min="0"
              placeholder="最多可选"
              @input="$set(activeData, 'max', $event ? $event : undefined)"/>
          </el-form-item>
          <el-form-item v-if="activeData.prepend !== undefined" label="前缀">
            <el-input v-model="activeData.prepend" placeholder="请输入前缀" />
          </el-form-item>
          <el-form-item v-if="activeData.append !== undefined" label="后缀">
            <el-input v-model="activeData.append" placeholder="请输入后缀" />
          </el-form-item>
          <el-form-item v-if="activeData['prefix-icon'] !== undefined" label="前图标">
            <el-input v-model="activeData['prefix-icon']" placeholder="请输入前图标名称">
              <el-button
                slot="append"
                icon="el-icon-thumb"
                @click="openIconsDialog('prefix-icon')">
                选择
              </el-button>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData['suffix-icon'] !== undefined" label="后图标">
            <el-input v-model="activeData['suffix-icon']" placeholder="请输入后图标名称">
              <el-button
                slot="append"
                icon="el-icon-thumb"
                @click="openIconsDialog('suffix-icon')">
                选择
              </el-button>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-cascader'" label="选项分隔符">
            <el-input v-model="activeData.separator" placeholder="请输入选项分隔符" />
          </el-form-item>
          <el-form-item v-if="activeData.autosize !== undefined" label="最小行数">
            <el-input-number
              v-model="activeData.autosize.minRows"
              :min="1"
              placeholder="最小行数"/>
          </el-form-item>
          <el-form-item v-if="activeData.autosize !== undefined" label="最大行数">
            <el-input-number
              v-model="activeData.autosize.maxRows"
              :min="1"
              placeholder="最大行数"/>
          </el-form-item>
          <el-form-item v-if="activeData.min !== undefined" label="最小值">
            <el-input-number v-model="activeData.min" placeholder="最小值" />
          </el-form-item>
          <el-form-item v-if="activeData.max !== undefined" label="最大值">
            <el-input-number v-model="activeData.max" placeholder="最大值" />
          </el-form-item>
          <el-form-item v-if="activeData.step !== undefined" label="步长">
            <el-input-number v-model="activeData.step" placeholder="步数" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-input-number'" label="精度">
            <el-input-number
              v-model="activeData.precision"
              :min="0"
              placeholder="精度"/>
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-input-number'" label="按钮位置">
            <el-radio-group v-model="activeData['controls-position']">
              <el-radio-button label=""> 默认 </el-radio-button>
              <el-radio-button label="right"> 右侧 </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData.maxlength !== undefined" label="最多输入">
            <el-input v-model="activeData.maxlength" placeholder="请输入字符长度">
              <template slot="append"> 个字符 </template>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData['active-text'] !== undefined" label="开启提示">
            <el-input v-model="activeData['active-text']" placeholder="请输入开启提示" />
          </el-form-item>
          <el-form-item v-if="activeData['inactive-text'] !== undefined" label="关闭提示">
            <el-input v-model="activeData['inactive-text']" placeholder="请输入关闭提示" />
          </el-form-item>
          <el-form-item v-if="activeData['active-value'] !== undefined" label="开启值">
            <el-input
              :value="setDefaultValue(activeData['active-value'])"
              placeholder="请输入开启值"
              @input="onSwitchValueInput($event, 'active-value')"/>
          </el-form-item>
          <el-form-item v-if="activeData['inactive-value'] !== undefined" label="关闭值">
            <el-input
              :value="setDefaultValue(activeData['inactive-value'])"
              placeholder="请输入关闭值"
              @input="onSwitchValueInput($event, 'inactive-value')"/>
          </el-form-item>
          <el-form-item
            v-if="activeData.type !== undefined && 'el-date-picker' === activeData.tag"
            label="时间类型">
            <el-select
              v-model="activeData.type"
              placeholder="请选择时间类型"
              :style="{ width: '100%' }"
              @change="dateTypeChange">
              <el-option
                v-for="(item, index) in dateOptions"
                :key="index"
                :label="item.label"
                :value="item.value"/>
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.name !== undefined" label="文件字段名">
            <el-input v-model="activeData.name" placeholder="请输入上传文件字段名" />
          </el-form-item>
          <el-form-item v-if="activeData.accept !== undefined" label="文件类型">
            <el-select
              v-model="activeData.accept"
              placeholder="请选择文件类型"
              :style="{ width: '100%' }"
              clearable>
              <el-option label="图片" value="image/*" />
              <el-option label="视频" value="video/*" />
              <el-option label="音频" value="audio/*" />
              <el-option label="excel" value=".xls,.xlsx" />
              <el-option label="word" value=".doc,.docx" />
              <el-option label="pdf" value=".pdf" />
              <el-option label="txt" value=".txt" />
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.fileSize !== undefined" label="文件大小">
            <el-input v-model.number="activeData.fileSize" placeholder="请输入文件大小">
              <el-select
                slot="append"
                v-model="activeData.sizeUnit"
                :style="{ width: '66px' }">
                <el-option label="KB" value="KB" />
                <el-option label="MB" value="MB" />
                <el-option label="GB" value="GB" />
              </el-select>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData.action !== undefined" label="上传地址">
            <el-input
              v-model="activeData.action"
              placeholder="请输入上传地址"
              clearable/>
          </el-form-item>
          <el-form-item v-if="activeData['list-type'] !== undefined" label="列表类型">
            <el-radio-group v-model="activeData['list-type']" size="small">
              <el-radio-button label="text"> text </el-radio-button>
              <el-radio-button label="picture"> picture </el-radio-button>
              <el-radio-button label="picture-card"> picture-card </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            v-if="activeData.buttonText !== undefined"
            v-show="'picture-card' !== activeData['list-type']"
            label="按钮文字">
            <el-input v-model="activeData.buttonText" placeholder="请输入按钮文字" />
          </el-form-item>
          <el-form-item v-if="activeData['range-separator'] !== undefined" label="分隔符">
            <el-input v-model="activeData['range-separator']" placeholder="请输入分隔符" />
          </el-form-item>
          <el-form-item v-if="activeData['picker-options'] !== undefined" label="时间段">
            <el-input v-model="activeData['picker-options'].selectableRange" placeholder="请输入时间段" />
          </el-form-item>
          <el-form-item v-if="activeData.format !== undefined" label="时间格式">
            <el-input
              :value="activeData.format"
              placeholder="请输入时间格式"
              @input="setTimeValue($event)"/>
          </el-form-item>
          <template
            v-if="['el-checkbox-group', 'el-radio-group', 'el-select'].indexOf(activeData.tag) > -1">
            <el-divider>选项</el-divider>
            <draggable
              :list="activeData.options"
              :animation="340"
              group="selectItem"
              handle=".option-drag">
              <div
                v-for="(item, index) in activeData.options"
                :key="index"
                class="select-item">
                <div class="select-line-icon option-drag">
                  <i class="el-icon-s-operation" />
                </div>
                <el-input
                  v-model="item.label"
                  placeholder="选项名"
                  size="small"/>
                <el-input
                  placeholder="选项值"
                  size="small"
                  :value="item.value"
                  @input="setOptionValue(item, $event)"/>
                <div
                  class="close-btn select-line-icon"
                  @click="activeData.options.splice(index, 1)">
                  <i class="el-icon-remove-outline" />
                </div>
              </div>
            </draggable>
            <div style="margin-left: 20px">
              <el-button
                style="padding-bottom: 0"
                icon="el-icon-circle-plus-outline"
                type="text"
                @click="addSelectItem">
                添加选项
              </el-button>
            </div>
            <el-divider />
          </template>

          <template v-if="['el-cascader'].indexOf(activeData.tag) > -1">
            <el-divider>选项</el-divider>
            <el-form-item label="数据类型">
              <el-radio-group v-model="activeData.dataType" size="small">
                <el-radio-button label="dynamic"> 动态数据 </el-radio-button>
                <el-radio-button label="static"> 静态数据 </el-radio-button>
              </el-radio-group>
            </el-form-item>

            <template v-if="activeData.dataType === 'dynamic'">
              <el-form-item label="标签键名">
                <el-input v-model="activeData.labelKey" placeholder="请输入标签键名" />
              </el-form-item>
              <el-form-item label="值键名">
                <el-input v-model="activeData.valueKey" placeholder="请输入值键名" />
              </el-form-item>
              <el-form-item label="子级键名">
                <el-input v-model="activeData.childrenKey" placeholder="请输入子级键名" />
              </el-form-item>
            </template>

            <el-tree
              v-if="activeData.dataType === 'static'"
              draggable
              :data="activeData.options"
              node-key="id"
              :expand-on-click-node="false"
              :render-content="renderContent"/>
            <div v-if="activeData.dataType === 'static'" style="margin-left: 20px">
              <el-button
                style="padding-bottom: 0"
                icon="el-icon-circle-plus-outline"
                type="text"
                @click="addTreeItem">
                添加父级
              </el-button>
            </div>
            <el-divider />
          </template>

          <el-form-item v-if="activeData.optionType !== undefined" label="选项样式">
            <el-radio-group v-model="activeData.optionType">
              <el-radio-button label="default"> 默认 </el-radio-button>
              <el-radio-button label="button"> 按钮 </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData['active-color'] !== undefined" label="开启颜色">
            <el-color-picker v-model="activeData['active-color']" />
          </el-form-item>
          <el-form-item v-if="activeData['inactive-color'] !== undefined" label="关闭颜色">
            <el-color-picker v-model="activeData['inactive-color']" />
          </el-form-item>

          <el-form-item v-if="activeData['allow-half'] !== undefined" label="允许半选">
            <el-switch v-model="activeData['allow-half']" />
          </el-form-item>
          <el-form-item v-if="activeData['show-text'] !== undefined" label="辅助文字">
            <el-switch v-model="activeData['show-text']" @change="rateTextChange" />
          </el-form-item>
          <el-form-item v-if="activeData['show-score'] !== undefined" label="显示分数">
            <el-switch v-model="activeData['show-score']" @change="rateScoreChange" />
          </el-form-item>
          <el-form-item v-if="activeData['show-stops'] !== undefined" label="显示间断点">
            <el-switch v-model="activeData['show-stops']" />
          </el-form-item>
          <el-form-item v-if="activeData.range !== undefined" label="范围选择">
            <el-switch v-model="activeData.range" @change="rangeChange" />
          </el-form-item>
          <el-form-item
            v-if="activeData.border !== undefined && activeData.optionType === 'default'"
            label="是否带边框">
            <el-switch v-model="activeData.border" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-color-picker'" label="颜色格式">
            <el-select
              v-model="activeData['color-format']"
              placeholder="请选择颜色格式"
              :style="{ width: '100%' }"
              @change="colorFormatChange">
              <el-option
                v-for="(item, index) in colorFormatOptions"
                :key="index"
                :label="item.label"
                :value="item.value"/>
            </el-select>
          </el-form-item>
          <el-form-item
            v-if="
              activeData.size !== undefined &&
              (activeData.optionType === 'button' ||
                activeData.border ||
                activeData.tag === 'el-color-picker')
            "
            label="选项尺寸">
            <el-radio-group v-model="activeData.size">
              <el-radio-button label="medium"> 中等 </el-radio-button>
              <el-radio-button label="small"> 较小 </el-radio-button>
              <el-radio-button label="mini"> 迷你 </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData['show-word-limit'] !== undefined" label="输入统计">
            <el-switch v-model="activeData['show-word-limit']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-input-number'" label="严格步数">
            <el-switch v-model="activeData['step-strictly']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-cascader'" label="是否多选">
            <el-switch v-model="activeData.props.props.multiple" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-cascader'" label="展示全路径">
            <el-switch v-model="activeData['show-all-levels']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-cascader'" label="可否筛选">
            <el-switch v-model="activeData.filterable" />
          </el-form-item>
          <el-form-item v-if="activeData.clearable !== undefined" label="能否清空">
            <el-switch v-model="activeData.clearable" />
          </el-form-item>
          <el-form-item v-if="activeData.showTip !== undefined" label="显示提示">
            <el-switch v-model="activeData.showTip" />
          </el-form-item>
          <el-form-item v-if="activeData.multiple !== undefined" label="多选文件">
            <el-switch v-model="activeData.multiple" />
          </el-form-item>
          <el-form-item v-if="activeData['auto-upload'] !== undefined" label="自动上传">
            <el-switch v-model="activeData['auto-upload']" />
          </el-form-item>
          <el-form-item v-if="activeData.readonly !== undefined" label="是否只读">
            <el-switch v-model="activeData.readonly" />
          </el-form-item>
          <el-form-item v-if="activeData.disabled !== undefined" label="是否禁用">
            <el-switch v-model="activeData.disabled" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-select'" label="是否可搜索">
            <el-switch v-model="activeData.filterable" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-select'" label="是否多选">
            <el-switch v-model="activeData.multiple" @change="multipleChange" />
          </el-form-item>
          <el-form-item v-if="activeData.required !== undefined" label="是否必填">
            <el-switch v-model="activeData.required" />
          </el-form-item>

          <template v-if="activeData.layoutTree">
            <el-divider>布局结构树</el-divider>
            <el-tree
              :data="[activeData]"
              :props="layoutTreeProps"
              node-key="renderKey"
              default-expand-all
              draggable>
              <span slot-scope="{ node, data }">
                <span class="node-label">
                  <svg-icon class="node-icon" :icon-class="data.tagIcon" />
                  {{ node.label }}
                </span>
              </span>
            </el-tree>
          </template>

          <template v-if="activeData.layout === 'colFormItem' && activeData.tag !== 'el-button'">
            <el-divider>正则校验</el-divider>
            <div
              v-for="(item, index) in activeData.regList"
              :key="index"
              class="reg-item">
              <span class="close-btn" @click="activeData.regList.splice(index, 1)">
                <i class="el-icon-close" />
              </span>
              <el-form-item label="表达式">
                <el-input v-model="item.pattern" placeholder="请输入正则" />
              </el-form-item>
              <el-form-item label="错误提示" style="margin-bottom: 0">
                <el-input v-model="item.message" placeholder="请输入错误提示" />
              </el-form-item>
            </div>
            <div style="margin-left: 20px">
              <el-button
                icon="el-icon-circle-plus-outline"
                type="text"
                @click="addReg">
                添加规则
              </el-button>
            </div>
          </template>
        </el-form>
        <!-- 表单属性 -->
        <el-form
          v-show="currentTab === 'form'"
          size="small"
          label-width="90px">
          <el-form-item label="表单名">
            <el-input v-model="formConf.formRef" placeholder="请输入表单名（ref）" />
          </el-form-item>
          <el-form-item label="表单模型">
            <el-input v-model="formConf.formModel" placeholder="请输入数据模型" />
          </el-form-item>
          <el-form-item label="校验模型">
            <el-input v-model="formConf.formRules" placeholder="请输入校验模型" />
          </el-form-item>
          <el-form-item label="表单尺寸">
            <el-radio-group v-model="formConf.size">
              <el-radio-button label="medium"> 中等 </el-radio-button>
              <el-radio-button label="small"> 较小 </el-radio-button>
              <el-radio-button label="mini"> 迷你 </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="标签对齐">
            <el-radio-group v-model="formConf.labelPosition">
              <el-radio-button label="left"> 左对齐 </el-radio-button>
              <el-radio-button label="right"> 右对齐 </el-radio-button>
              <el-radio-button label="top"> 顶部对齐 </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="标签宽度">
            <el-input-number v-model="formConf.labelWidth" placeholder="标签宽度" />
          </el-form-item>
          <el-form-item label="栅格间隔">
            <el-input-number
              v-model="formConf.gutter"
              :min="0"
              placeholder="栅格间隔"/>
          </el-form-item>
          <el-form-item label="禁用表单">
            <el-switch v-model="formConf.disabled" />
          </el-form-item>
          <el-form-item label="表单按钮">
            <el-switch v-model="formConf.formBtns" />
          </el-form-item>
          <el-form-item label="显示未选中组件边框">
            <el-switch v-model="formConf.unFocusedComponentBorder" />
          </el-form-item>
        </el-form>
      </el-scrollbar>
    </div>

    <treeNode-dialog
      :visible.sync="dialogVisible"
      title="添加选项"
      @commit="addNode"/>
    <icons-dialog
      :visible.sync="iconsVisible"
      :current="activeData[currentIconModel]"
      @select="setIcon"/>
  </div>
</template>

<script>
/**
 * 组件名称：RightPanel
 * 功能说明：表单生成器右侧属性配置面板，用于配置选中组件与表单的各类属性
 * 研发时间：2024/12/11
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} showField  - 是否显示组件属性面板
 * @prop {Object}  activeData - 当前选中的组件配置对象
 * @prop {Object}  formConf   - 表单全局配置对象
 *
 * 变量说明：
 * @data {String} currentTab            - 当前选中的页签（field/form）
 * @data {Object} currentNode           - 当前操作的树节点
 * @data {Boolean} dialogVisible        - 树形选项弹窗是否可见
 * @data {Boolean} iconsVisible         - 图标选择弹窗是否可见
 * @data {String} currentIconModel      - 当前操作的图标字段名
 * @data {Array}  dateTypeOptions       - 日期类型选项
 * @data {Array}  dateRangeTypeOptions  - 日期范围类型选项
 * @data {Array}  colorFormatOptions    - 颜色格式选项
 * @data {Array}  justifyOptions        - 水平排列选项
 * @data {Object} layoutTreeProps       - 布局结构树 props 配置
 *
 * 事件说明：
 * @event addReg              - 添加一条正则校验规则
 * @event addSelectItem       - 添加一个下拉选项
 * @event addTreeItem         - 打开弹窗以添加树形父级选项
 * @event renderContent       - 自定义渲染树节点内容
 * @event append              - 在指定节点下添加子级
 * @event remove              - 删除指定子级节点
 * @event addNode             - 添加树节点数据
 * @event setOptionValue      - 设置下拉选项值
 * @event setDefaultValue     - 格式化默认值用于展示
 * @event onDefaultValueInput - 处理默认值输入
 * @event onSwitchValueInput  - 处理开关自定义值输入
 * @event setTimeValue        - 设置日期组件默认值与格式
 * @event spanChange          - 更新表单栅格 span
 * @event multipleChange      - 处理多选切换重置默认值
 * @event dateTypeChange      - 处理日期类型切换
 * @event rangeChange         - 处理范围选择切换重置默认值
 * @event rateTextChange      - 处理评分辅助文字开关
 * @event rateScoreChange     - 处理评分显示分数开关
 * @event colorFormatChange   - 处理颜色格式切换
 * @event openIconsDialog     - 打开图标选择弹窗
 * @event setIcon             - 设置选中图标
 * @event tagChange           - 处理组件类型切换
 *
 * 插槽说明：
 */

/**
 * 2024/12/11     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { isArray } from 'util'
import draggable from 'vuedraggable'
import TreeNodeDialog from './TreeNodeDialog'
import { isNumberStr } from '@/utils/index'
import IconsDialog from './IconsDialog'
import { inputComponents, selectComponents, layoutComponents } from '@/utils/generator/config'

const dateTimeFormat = {
  date: 'yyyy-MM-dd',
  week: 'yyyy 第 WW 周',
  month: 'yyyy-MM',
  year: 'yyyy',
  datetime: 'yyyy-MM-dd HH:mm:ss',
  daterange: 'yyyy-MM-dd',
  monthrange: 'yyyy-MM',
  datetimerange: 'yyyy-MM-dd HH:mm:ss',
}

export default
{
  components: {
    draggable,
    TreeNodeDialog,
    IconsDialog,
  },

  props: ['showField', 'activeData', 'formConf'],

  data()
  {
    return {
      currentTab: 'field',
      currentNode: null,
      dialogVisible: false,
      iconsVisible: false,
      currentIconModel: null,
      dateTypeOptions: [
        {
          label: '日(date)',
          value: 'date',
        },
        {
          label: '周(week)',
          value: 'week',
        },
        {
          label: '月(month)',
          value: 'month',
        },
        {
          label: '年(year)',
          value: 'year',
        },
        {
          label: '日期时间(datetime)',
          value: 'datetime',
        },
      ],
      dateRangeTypeOptions: [
        {
          label: '日期范围(daterange)',
          value: 'daterange',
        },
        {
          label: '月范围(monthrange)',
          value: 'monthrange',
        },
        {
          label: '日期时间范围(datetimerange)',
          value: 'datetimerange',
        },
      ],
      colorFormatOptions: [
        {
          label: 'hex',
          value: 'hex',
        },
        {
          label: 'rgb',
          value: 'rgb',
        },
        {
          label: 'rgba',
          value: 'rgba',
        },
        {
          label: 'hsv',
          value: 'hsv',
        },
        {
          label: 'hsl',
          value: 'hsl',
        },
      ],
      justifyOptions: [
        {
          label: 'start',
          value: 'start',
        },
        {
          label: 'end',
          value: 'end',
        },
        {
          label: 'center',
          value: 'center',
        },
        {
          label: 'space-around',
          value: 'space-around',
        },
        {
          label: 'space-between',
          value: 'space-between',
        },
      ],
      layoutTreeProps: {
        label(data, node) {
          return data.componentName || `${data.label}: ${data.vModel}`
        },
      },
    }
  },

  computed:
  {
    /**
     * 当前组件的文档地址
     * @returns {String} 文档地址
     */
    documentLink()
    {
      return this.activeData.document || 'https://element.eleme.cn/#/zh-CN/component/installation'
    },

    /**
     * 根据日期组件配置返回可选的时间类型
     * @returns {Array} 时间类型选项
     */
    dateOptions()
    {
      if (this.activeData.type !== undefined && this.activeData.tag === 'el-date-picker')
      {
        if (this.activeData['start-placeholder'] === undefined)
        {
          return this.dateTypeOptions
        }
        return this.dateRangeTypeOptions
      }
      return []
    },

    /**
     * 组件类型分组列表
     * @returns {Array} 组件类型分组
     */
    tagList()
    {
      return [
        {
          label: '输入型组件',
          options: inputComponents,
        },
        {
          label: '选择型组件',
          options: selectComponents,
        },
      ]
    },
  },

  methods:
  {
    /**
     * 添加一条正则校验规则
     */
    addReg()
    {
      this.activeData.regList.push({
        pattern: '',
        message: '',
      })
    },

    /**
     * 添加一个下拉选项
     */
    addSelectItem()
    {
      this.activeData.options.push({
        label: '',
        value: '',
      })
    },

    /**
     * 打开弹窗以添加树形父级选项
     */
    addTreeItem()
    {
      ++this.idGlobal
      this.dialogVisible = true
      this.currentNode = this.activeData.options
    },

    /**
     * 自定义渲染树节点内容
     * @param {Function} h - createElement 函数
     * @param {Object} params - 节点参数（node/data/store）
     * @returns {VNode} 渲染的虚拟节点
     */
    renderContent(h, { node, data, store })
    {
      return (
        <div class="custom-tree-node">
          <span>{node.label}</span>
          <span class="node-operation">
            <i
              on-click={() => this.append(data)}
              class="el-icon-plus"
              title="添加"
            ></i>
            <i
              on-click={() => this.remove(node, data)}
              class="el-icon-delete"
              title="删除"
            ></i>
          </span>
        </div>
      )
    },

    /**
     * 在指定节点下添加子级
     * @param {Object} data - 节点数据
     */
    append(data)
    {
      if (!data.children)
      {
        this.$set(data, 'children', [])
      }
      this.dialogVisible = true
      this.currentNode = data.children
    },

    /**
     * 删除指定子级节点
     * @param {Object} node - 当前节点
     * @param {Object} data - 节点数据
     */
    remove(node, data)
    {
      const { parent } = node
      const children = parent.data.children || parent.data
      const index = children.findIndex((d) => d.id === data.id)
      children.splice(index, 1)
    },

    /**
     * 添加树节点数据
     * @param {Object} data - 节点数据
     */
    addNode(data)
    {
      this.currentNode.push(data)
    },

    /**
     * 设置下拉选项值
     * @param {Object} item - 选项对象
     * @param {String} val - 选项值
     */
    setOptionValue(item, val)
    {
      item.value = isNumberStr(val) ? +val : val
    },

    /**
     * 格式化默认值用于展示
     * @param {*} val - 原始默认值
     * @returns {*} 格式化后的默认值
     */
    setDefaultValue(val)
    {
      if (Array.isArray(val))
      {
        return val.join(',')
      }
      if (['string', 'number'].indexOf(val) > -1)
      {
        return val
      }
      if (typeof val === 'boolean')
      {
        return `${val}`
      }
      return val
    },

    /**
     * 处理默认值输入
     * @param {String} str - 输入字符串
     */
    onDefaultValueInput(str)
    {
      if (isArray(this.activeData.defaultValue))
      {
        // 数组
        this.$set(
          this.activeData,
          'defaultValue',
          str.split(',').map((val) => (isNumberStr(val) ? +val : val))
        )
      }
      else if (['true', 'false'].indexOf(str) > -1)
      {
        // 布尔
        this.$set(this.activeData, 'defaultValue', JSON.parse(str))
      }
      else
      {
        // 字符串和数字
        this.$set(this.activeData, 'defaultValue', isNumberStr(str) ? +str : str)
      }
    },

    /**
     * 处理开关自定义值输入
     * @param {String} val - 输入值
     * @param {String} name - 字段名
     */
    onSwitchValueInput(val, name)
    {
      if (['true', 'false'].indexOf(val) > -1)
      {
        this.$set(this.activeData, name, JSON.parse(val))
      }
      else
      {
        this.$set(this.activeData, name, isNumberStr(val) ? +val : val)
      }
    },

    /**
     * 设置日期组件默认值与格式
     * @param {String} val - 日期格式
     * @param {String} type - 日期类型
     */
    setTimeValue(val, type)
    {
      const valueFormat = type === 'week' ? dateTimeFormat.date : val
      this.$set(this.activeData, 'defaultValue', null)
      this.$set(this.activeData, 'value-format', valueFormat)
      this.$set(this.activeData, 'format', val)
    },

    /**
     * 更新表单栅格 span
     * @param {Number} val - 栅格值
     */
    spanChange(val)
    {
      this.formConf.span = val
    },

    /**
     * 处理多选切换重置默认值
     * @param {Boolean} val - 是否多选
     */
    multipleChange(val)
    {
      this.$set(this.activeData, 'defaultValue', val ? [] : '')
    },

    /**
     * 处理日期类型切换
     * @param {String} val - 日期类型
     */
    dateTypeChange(val)
    {
      this.setTimeValue(dateTimeFormat[val], val)
    },

    /**
     * 处理范围选择切换重置默认值
     * @param {Boolean} val - 是否范围选择
     */
    rangeChange(val)
    {
      this.$set(
        this.activeData,
        'defaultValue',
        val ? [this.activeData.min, this.activeData.max] : this.activeData.min
      )
    },

    /**
     * 处理评分辅助文字开关
     * @param {Boolean} val - 是否开启
     */
    rateTextChange(val)
    {
      if (val) this.activeData['show-score'] = false
    },

    /**
     * 处理评分显示分数开关
     * @param {Boolean} val - 是否开启
     */
    rateScoreChange(val)
    {
      if (val) this.activeData['show-text'] = false
    },

    /**
     * 处理颜色格式切换
     * @param {String} val - 颜色格式
     */
    colorFormatChange(val)
    {
      this.activeData.defaultValue = null
      this.activeData['show-alpha'] = val.indexOf('a') > -1
      this.activeData.renderKey = +new Date() // 更新renderKey,重新渲染该组件
    },

    /**
     * 打开图标选择弹窗
     * @param {String} model - 图标字段名
     */
    openIconsDialog(model)
    {
      this.iconsVisible = true
      this.currentIconModel = model
    },

    /**
     * 设置选中图标
     * @param {String} val - 图标名称
     */
    setIcon(val)
    {
      this.activeData[this.currentIconModel] = val
    },

    /**
     * 处理组件类型切换
     * @param {String} tagIcon - 组件图标名
     */
    tagChange(tagIcon)
    {
      let target = inputComponents.find((item) => item.tagIcon === tagIcon)
      if (!target) target = selectComponents.find((item) => item.tagIcon === tagIcon)
      this.$emit('tag-change', target)
    },
  },
}
</script>