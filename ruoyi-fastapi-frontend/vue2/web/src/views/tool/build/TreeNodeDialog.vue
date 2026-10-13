<style lang="scss" scoped></style>

<template>
  <div>
    <el-dialog
      :close-on-click-modal="false"
      :modal-append-to-body="false"
      v-bind="$attrs"
      v-on="$listeners"
      @open="onOpen"
      @close="onClose">
      <el-row :gutter="0">
        <el-form
          ref="elForm"
          :model="formData"
          :rules="rules"
          size="small"
          label-width="100px">
          <el-col :span="24">
            <el-form-item label="选项名" prop="label">
              <el-input
                v-model="formData.label"
                placeholder="请输入选项名"
                clearable/>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="选项值" prop="value">
              <el-input
                v-model="formData.value"
                placeholder="请输入选项值"
                clearable>
                <el-select
                  slot="append"
                  v-model="dataType"
                  :style="{ width: '100px' }">
                  <el-option
                    v-for="(item, index) in dataTypeOptions"
                    :key="index"
                    :label="item.label"
                    :value="item.value"
                    :disabled="item.disabled"/>
                </el-select>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
      <div slot="footer">
        <el-button type="primary" @click="handleConfirm">
          确定
        </el-button>
        <el-button @click="close"> 取消 </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
/**
 * 组件名称：TreeNodeDialog
 * 功能说明：树形节点选项弹窗组件，用于配置选项的选项名与选项值
 * 研发时间：2024/12/11
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {Number} id              - 选项自增编号
 * @data {Object} formData        - 表单数据（label 选项名 / value 选项值）
 * @data {Object} rules           - 表单校验规则
 * @data {String} dataType        - 选项值数据类型（string/number）
 * @data {Array}  dataTypeOptions - 选项值数据类型下拉选项
 *
 * 事件说明：
 * @event onOpen        - 弹窗打开时重置表单数据
 * @event onClose       - 弹窗关闭回调
 * @event close         - 关闭弹窗
 * @event handleConfirm - 校验通过后提交选项数据
 *
 * 插槽说明：
 */

/**
 * 2024/12/11     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { isNumberStr } from '@/utils/index'

export default
{
  components: {},

  inheritAttrs: false,

  props: [],

  data()
  {
    return {
      id: 100,
      formData: {
        label: undefined,
        value: undefined,
      },
      rules: {
        label: [
          {
            required: true,
            message: '请输入选项名',
            trigger: 'blur',
          },
        ],
        value: [
          {
            required: true,
            message: '请输入选项值',
            trigger: 'blur',
          },
        ],
      },
      dataType: 'string',
      dataTypeOptions: [
        {
          label: '字符串',
          value: 'string',
        },
        {
          label: '数字',
          value: 'number',
        },
      ],
    }
  },

  computed: {},

  watch:
  {
    /**
     * 监听选项值变化，自动切换数据类型
     * @param {String} val - 选项值
     */
    'formData.value': function (val)
    {
      this.dataType = isNumberStr(val) ? 'number' : 'string'
    },
  },

  /**
   * 组件创建完成
   */
  created() {},

  /**
   * 组件挂载完成
   */
  mounted() {},

  methods:
  {
    /**
     * 弹窗打开时重置表单数据
     */
    onOpen()
    {
      this.formData = {
        label: undefined,
        value: undefined,
      }
    },

    /**
     * 弹窗关闭回调
     */
    onClose() {},

    /**
     * 关闭弹窗
     */
    close()
    {
      this.$emit('update:visible', false)
    },

    /**
     * 校验通过后提交选项数据
     */
    handleConfirm()
    {
      this.$refs.elForm.validate((valid) =>
      {
        if (!valid) return
        if (this.dataType === 'number')
        {
          this.formData.value = parseFloat(this.formData.value)
        }
        this.formData.id = this.id++
        this.$emit('commit', this.formData)
        this.close()
      })
    },
  },
}
</script>