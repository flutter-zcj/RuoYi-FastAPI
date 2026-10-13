<style lang="scss" scoped></style>

<template>
  <div>
    <el-dialog
      width="500px"
      :close-on-click-modal="false"
      :modal-append-to-body="false"
      v-bind="$attrs"
      v-on="$listeners"
      @open="onOpen"
      @close="onClose">
      <el-row :gutter="15">
        <el-form
          ref="elForm"
          :model="formData"
          :rules="rules"
          size="medium"
          label-width="100px">
          <el-col :span="24">
            <el-form-item label="生成类型" prop="type">
              <el-radio-group v-model="formData.type">
                <el-radio-button
                  v-for="(item, index) in typeOptions"
                  :key="index"
                  :label="item.value"
                  :disabled="item.disabled">
                  {{ item.label }}
                </el-radio-button>
              </el-radio-group>
            </el-form-item>
            <el-form-item
              v-if="showFileName"
              label="文件名"
              prop="fileName">
              <el-input
                v-model="formData.fileName"
                placeholder="请输入文件名"
                clearable/>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>

      <div slot="footer">
        <el-button @click="close"> 取消 </el-button>
        <el-button type="primary" @click="handleConfirm">
          确定
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
/**
 * 组件名称：CodeTypeDialog
 * 功能说明：代码生成类型选择弹窗组件，用于选择生成页面或弹窗
 * 研发时间：2024/12/11
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} showFileName - 是否显示文件名输入项
 *
 * 变量说明：
 * @data {Object} formData    - 表单数据（fileName 文件名 / type 生成类型）
 * @data {Object} rules       - 表单校验规则
 * @data {Array}  typeOptions - 生成类型下拉选项
 *
 * 事件说明：
 * @event onOpen        - 弹窗打开时初始化文件名
 * @event onClose       - 弹窗关闭回调
 * @event close         - 关闭弹窗
 * @event handleConfirm - 校验通过后提交生成配置
 *
 * 插槽说明：
 */

/**
 * 2024/12/11     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  inheritAttrs: false,

  props: ['showFileName'],

  data()
  {
    return {
      formData: {
        fileName: undefined,
        type: 'file',
      },
      rules: {
        fileName: [
          {
            required: true,
            message: '请输入文件名',
            trigger: 'blur',
          },
        ],
        type: [
          {
            required: true,
            message: '生成类型不能为空',
            trigger: 'change',
          },
        ],
      },
      typeOptions: [
        {
          label: '页面',
          value: 'file',
        },
        {
          label: '弹窗',
          value: 'dialog',
        },
      ],
    }
  },

  computed: {},

  watch: {},

  /**
   * 组件挂载完成
   */
  mounted() {},

  methods:
  {
    /**
     * 弹窗打开时初始化文件名
     */
    onOpen()
    {
      if (this.showFileName)
      {
        this.formData.fileName = `${+new Date()}.vue`
      }
    },

    /**
     * 弹窗关闭回调
     */
    onClose() {},

    /**
     * 关闭弹窗
     * @param {Event} e - 事件对象
     */
    close(e)
    {
      this.$emit('update:visible', false)
    },

    /**
     * 校验通过后提交生成配置
     */
    handleConfirm()
    {
      this.$refs.elForm.validate((valid) =>
      {
        if (!valid) return
        this.$emit('confirm', { ...this.formData })
        this.close()
      })
    },
  },
}
</script>