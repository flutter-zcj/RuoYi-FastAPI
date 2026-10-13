<template>
  <el-dialog
    :title="title"
    :visible.sync="visible"
    :width="width"
    append-to-body
    @close="handleClose">
    <el-upload
      ref="uploadRef"
      :limit="1"
      accept=".xlsx, .xls"
      :headers="headers"
      :action="uploadUrl"
      :disabled="isUploading"
      :on-progress="handleProgress"
      :on-success="handleSuccess"
      :auto-upload="false"
      drag>
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div
        class="el-upload__tip text-center"
        slot="tip">
        <div
          class="el-upload__tip"
          slot="tip">
          <el-checkbox v-model="updateSupport"> {{ updateSupportLabel }} </el-checkbox>
        </div>
        <span>仅允许导入xls、xlsx格式文件。</span>
        <el-link
          type="primary"
          :underline="false"
          style="font-size: 12px; vertical-align: baseline"
          v-if="templateUrl"
          @click="handleDownloadTemplate">下载模板</el-link>
      </div>
    </el-upload>
    <div
      class="dialog-footer"
      slot="footer">
      <el-button
        type="primary"
        @click="handleSubmit">确 定</el-button>
      <el-button @click="visible = false">取 消</el-button>
    </div>
  </el-dialog>
</template>

<script>
/**
 * 组件名称：ExcelImportDialog
 * 功能说明：Excel 导入对话框组件，支持文件上传、模板下载与导入结果提示
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {String} title              - 对话框标题
 * @prop {String} width              - 对话框宽度
 * @prop {String} action             - 上传接口地址（必传）
 * @prop {String} templateAction     - 模板下载接口地址，不传则不显示下载模板链接
 * @prop {String} templateFileName   - 模板文件名
 * @prop {String} updateSupportLabel - 覆盖更新勾选框的说明文字
 *
 * 变量说明：
 *
 * 事件说明：
 * @event success - 导入成功
 *
 * 插槽说明：
 */

/**
 * 2026/07/31     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getToken } from '@/utils/auth'

export default
{
  props:
  {
    // 对话框标题
    title: { type: String, default: '数据导入' },
    // 对话框宽度
    width: { type: String, default: '400px' },
    // 上传接口地址（必传）
    action: { type: String, required: true },
    // 模板下载接口地址，不传则不显示下载模板链接
    templateAction: { type: String, default: '' },
    // 模板文件名
    templateFileName: { type: String, default: 'template' },
    // 覆盖更新勾选框的说明文字
    updateSupportLabel: { type: String, default: '是否更新已经存在的数据' }
  },

  data()
  {
    return {
      visible: false,
      isUploading: false,
      updateSupport: false,
      headers: { Authorization: 'Bearer ' + getToken() },
    }
  },

  computed:
  {
    /**
     * 计算带更新支持参数的上传地址
     * @returns {String} 上传地址
     */
    uploadUrl()
    {
      return (
        process.env.VUE_APP_BASE_API +
        this.action +
        '?updateSupport=' +
        (this.updateSupport ? 1 : 0)
      )
    },

    /**
     * 是否显示下载模板链接
     * @returns {Boolean} 是否显示
     */
    templateUrl()
    {
      return !!this.templateAction
    }
  },

  methods:
  {
    /**
     * 打开对话框（供父组件通过 ref 调用）
     */
    open()
    {
      this.updateSupport = false
      this.isUploading = false
      this.visible = true
      this.$nextTick(() =>
      {
        if (this.$refs.uploadRef)
        {
          this.$refs.uploadRef.clearFiles()
        }
      })
    },

    /**
     * 关闭时清理上传状态
     */
    handleClose()
    {
      this.isUploading = false
      if (this.$refs.uploadRef)
      {
        this.$refs.uploadRef.clearFiles()
      }
    },

    /**
     * 下载导入模板
     */
    handleDownloadTemplate()
    {
      this.download(
        this.templateAction,
        {},
        `${this.templateFileName}_${new Date().getTime()}.xlsx`
      )
    },

    /**
     * 上传进度回调
     */
    handleProgress()
    {
      this.isUploading = true
    },

    /**
     * 上传成功回调
     * @param {Object} response - 接口返回结果
     */
    handleSuccess(response)
    {
      this.visible = false
      this.isUploading = false
      if (this.$refs.uploadRef)
      {
        this.$refs.uploadRef.clearFiles()
      }
      this.$alert(
        "<div style='overflow:auto;overflow-x:hidden;max-height:70vh;padding:10px 20px 0;'>" +
          response.msg +
          '</div>',
        '导入结果',
        { dangerouslyUseHTMLString: true }
      )
      this.$emit('success')
    },

    /**
     * 提交上传
     */
    handleSubmit()
    {
      const files = this.$refs.uploadRef.uploadFiles
      if (!files || files.length === 0)
      {
        this.$modal.msgError('请选择要上传的文件。')
        return
      }
      const name = files[0].name.toLowerCase()
      if (!name.endsWith('.xls') && !name.endsWith('.xlsx'))
      {
        this.$modal.msgError('请选择后缀为 "xls" 或 "xlsx" 的文件。')
        return
      }
      this.$refs.uploadRef.submit()
    }
  }
}
</script>