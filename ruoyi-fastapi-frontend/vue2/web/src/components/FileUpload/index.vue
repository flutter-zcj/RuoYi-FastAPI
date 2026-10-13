<style lang="scss" scoped>
.file-upload-darg { background: #c8ebfb; opacity: 0.5; }
.upload-file-uploader { margin-bottom: 5px; }
.upload-file-list .el-upload-list__item { position: relative; border: 1px solid #e4e7ed; margin-bottom: 10px; line-height: 2; }
.upload-file-list .ele-upload-list__item-content { display: flex; justify-content: space-between; align-items: center; color: inherit; }
.ele-upload-list__item-content-action .el-link { margin-right: 10px; }
</style>

<template>
  <div class="upload-file">
    <el-upload
      class="upload-file-uploader"
      ref="fileUpload"
      multiple
      :action="uploadFileUrl"
      :before-upload="handleBeforeUpload"
      :file-list="fileList"
      :data="data"
      :limit="limit"
      :on-error="handleUploadError"
      :on-exceed="handleExceed"
      :on-success="handleUploadSuccess"
      :show-file-list="false"
      :headers="headers"
      v-if="!disabled">
      <!-- 上传按钮 -->
      <el-button
        size="mini"
        type="primary">选取文件</el-button>
      <!-- 上传提示 -->
      <div
        class="el-upload__tip"
        slot="tip"
        v-if="showTip">
        请上传
        <template v-if="fileSize">
          大小不超过 <b style="color: #f56c6c">{{ fileSize }}MB</b>
        </template>
        <template v-if="fileType">
          格式为 <b style="color: #f56c6c">{{ fileType.join('/') }}</b>
        </template>
        的文件
      </div>
    </el-upload>

    <!-- 文件列表 -->
    <transition-group
      class="upload-file-list el-upload-list el-upload-list--text"
      ref="uploadFileList"
      name="el-fade-in-linear"
      tag="ul">
      <li
        class="el-upload-list__item ele-upload-list__item-content"
        :key="file.url"
        v-for="(file, index) in fileList">
        <el-link
          :href="`${baseUrl}${file.url}`"
          :underline="false"
          target="_blank"
          @click="handleFileDownload($event, file)">
          <span class="el-icon-document"> {{ getFileName(file.name) }} </span>
        </el-link>
        <div class="ele-upload-list__item-content-action">
          <el-link
            :underline="false"
            type="danger"
            v-if="!disabled"
            @click="handleDelete(index)">删除</el-link>
        </div>
      </li>
    </transition-group>
  </div>
</template>

<script>
/**
 * 组件名称：FileUpload
 * 功能说明：文件上传组件，支持多文件上传、格式大小校验、拖动排序与下载
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {[String, Object, Array]} value     - 值
 * @prop {String}                  action    - 上传接口地址
 * @prop {Boolean}                 isPrivate - 是否上传受保护文件
 * @prop {Object}                  data      - 上传携带的参数
 * @prop {Number}                  limit     - 数量限制
 * @prop {Number}                  fileSize  - 大小限制(MB)
 * @prop {Array}                   fileType  - 文件类型，例如['png', 'jpg', 'jpeg']
 * @prop {Boolean}                 isShowTip - 是否显示提示
 * @prop {Boolean}                 disabled  - 禁用组件（仅查看文件）
 * @prop {Boolean}                 drag      - 拖动排序
 *
 * 变量说明：
 *
 * 事件说明：
 * @event input - 文件列表变更
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getToken } from '@/utils/auth'
import Sortable from 'sortablejs'

export default
{
  name: 'FileUpload',

  props:
  {
    // 值
    value: [String, Object, Array],
    // 上传接口地址
    action: { type: String, default: '/common/upload' },
    // 是否上传受保护文件
    isPrivate: { type: Boolean, default: false },
    // 上传携带的参数
    data: { type: Object },
    // 数量限制
    limit: { type: Number, default: 5 },
    // 大小限制(MB)
    fileSize: { type: Number, default: 5 },
    // 文件类型, 例如['png', 'jpg', 'jpeg']
    fileType: { type: Array, default: () => ['doc', 'docx', 'xls', 'xlsx', 'ppt', 'pptx', 'txt', 'pdf'] },
    // 是否显示提示
    isShowTip: { type: Boolean, default: true },
    // 禁用组件（仅查看文件）
    disabled: { type: Boolean, default: false },
    // 拖动排序
    drag: { type: Boolean, default: true }
  },

  data()
  {
    const uploadAction =
      this.isPrivate && this.action === '/common/upload' ? '/common/files/upload' : this.action
    return {
      number: 0,
      uploadList: [],
      baseUrl: process.env.VUE_APP_BASE_API,
      uploadFileUrl: process.env.VUE_APP_BASE_API + uploadAction, // 上传文件服务器地址
      headers: {
        Authorization: 'Bearer ' + getToken(),
      },
      fileList: [],
    }
  },

  computed:
  {
    // 是否显示提示
    showTip()
    {
      return this.isShowTip && (this.fileType || this.fileSize)
    }
  },

  watch:
  {
    value:
    {
      /**
       * 监听值变化并转换为文件对象数组
       * @param {String|Object|Array} val - 新的值
       */
      handler(val)
      {
        if (val)
        {
          let temp = 1
          // 首先将值转为数组
          const list = Array.isArray(val) ? val : this.value.split(',')
          // 然后将数组转为对象数组
          this.fileList = list.map((item) =>
          {
            if (typeof item === 'string')
            {
              item = { name: item, url: item }
            }
            item.uid = item.uid || new Date().getTime() + temp++
            return item
          })
        }
        else
        {
          this.fileList = []
          return []
        }
      },

      deep: true,
      immediate: true
    }
  },

  mounted()
  {
    if (this.drag && !this.disabled)
    {
      this.$nextTick(() =>
      {
        const element = this.$refs.uploadFileList?.$el || this.$refs.uploadFileList
        Sortable.create(element, {
          ghostClass: 'file-upload-darg',
          onEnd: (evt) =>
          {
            const movedItem = this.fileList.splice(evt.oldIndex, 1)[0]
            this.fileList.splice(evt.newIndex, 0, movedItem)
            this.$emit('input', this.listToString(this.fileList))
          },
        })
      })
    }
  },

  methods:
  {
    /**
     * 上传前校验文件格式、文件名与大小
     * @param {Object} file - 待上传文件
     * @returns {Boolean} 是否允许上传
     */
    handleBeforeUpload(file)
    {
      // 校检文件类型
      if (this.fileType)
      {
        const fileName = file.name.split('.')
        const fileExt = fileName[fileName.length - 1]
        const isTypeOk = this.fileType.indexOf(fileExt) >= 0
        if (!isTypeOk)
        {
          this.$modal.msgError(`文件格式不正确，请上传${this.fileType.join('/')}格式文件!`)
          return false
        }
      }
      // 校检文件名是否包含特殊字符
      if (file.name.includes(','))
      {
        this.$modal.msgError('文件名不正确，不能包含英文逗号!')
        return false
      }
      // 校检文件大小
      if (this.fileSize)
      {
        const isLt = file.size / 1024 / 1024 < this.fileSize
        if (!isLt)
        {
          this.$modal.msgError(`上传文件大小不能超过 ${this.fileSize} MB!`)
          return false
        }
      }
      this.$modal.loading('正在上传文件，请稍候...')
      this.number++
      return true
    },

    /**
     * 文件个数超出限制提示
     */
    handleExceed()
    {
      this.$modal.msgError(`上传文件数量不能超过 ${this.limit} 个!`)
    },

    /**
     * 上传失败处理
     * @param {Object} err - 错误信息
     */
    handleUploadError(err)
    {
      this.$modal.msgError('上传文件失败，请重试')
      this.$modal.closeLoading()
    },

    /**
     * 上传成功回调
     * @param {Object} res  - 接口返回结果
     * @param {Object} file - 上传文件对象
     */
    handleUploadSuccess(res, file)
    {
      if (res.code === 200)
      {
        this.uploadList.push({ name: res.fileName, url: res.fileName })
        this.uploadedSuccessfully()
      }
      else
      {
        this.number--
        this.$modal.closeLoading()
        this.$modal.msgError(res.msg)
        this.$refs.fileUpload.handleRemove(file)
        this.uploadedSuccessfully()
      }
    },

    /**
     * 下载受保护文件
     * @param {Event}  event - 点击事件
     * @param {Object} file  - 文件对象
     */
    handleFileDownload(event, file)
    {
      if (typeof file.url === 'string' && file.url.startsWith('/common/files/'))
      {
        event.preventDefault()
        this.$download.file(file.url)
      }
    },

    /**
     * 删除文件
     * @param {Number} index - 文件索引
     */
    handleDelete(index)
    {
      this.fileList.splice(index, 1)
      this.$emit('input', this.listToString(this.fileList))
    },

    /**
     * 上传结束处理，全部完成时合并列表
     */
    uploadedSuccessfully()
    {
      if (this.number > 0 && this.uploadList.length === this.number)
      {
        this.fileList = this.fileList.concat(this.uploadList)
        this.uploadList = []
        this.number = 0
        this.$emit('input', this.listToString(this.fileList))
        this.$modal.closeLoading()
      }
    },

    /**
     * 获取文件名称
     * @param {String} name - 文件路径或名称
     * @returns {String} 文件名称
     */
    getFileName(name)
    {
      // 如果是url那么取最后的名字 如果不是直接返回
      if (name.lastIndexOf('/') > -1)
      {
        return name.slice(name.lastIndexOf('/') + 1)
      }
      else
      {
        return name
      }
    },

    /**
     * 对象转成指定字符串分隔
     * @param {Array}  list      - 文件列表
     * @param {String} separator - 分隔符
     * @returns {String} 拼接后的字符串
     */
    listToString(list, separator)
    {
      let strs = ''
      separator = separator || ','
      for (let i in list)
      {
        strs += list[i].url + separator
      }
      return strs != '' ? strs.substr(0, strs.length - 1) : ''
    }
  }
}
</script>