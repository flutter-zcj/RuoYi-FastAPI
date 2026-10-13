<style lang="scss" scoped>
.file-hash { font-family: monospace; word-break: break-all; }
</style>

<template>
  <el-dialog
    title="文件详细信息"
    :visible.sync="dialogOpen"
    width="820px"
    append-to-body>
    <el-descriptions :column="2" border>
      <el-descriptions-item label="文件ID" :span="2">
        {{ detail.fileId }}
      </el-descriptions-item>
      <el-descriptions-item label="原始文件名">
        {{ detail.originalName }}
      </el-descriptions-item>
      <el-descriptions-item label="存储文件名">
        {{ detail.storedName }}
      </el-descriptions-item>
      <el-descriptions-item label="访问类型">
        {{ accessTypeLabel(detail.accessType) }}
      </el-descriptions-item>
      <el-descriptions-item label="文件状态">
        {{ statusLabel(detail.status) }}
      </el-descriptions-item>
      <el-descriptions-item label="上传用户">
        {{ detail.createBy || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="上传用户ID">
        {{ detail.uploadUserId || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="上传人权限">
        <el-tag
          v-if="detail.accessType === 'private' && detail.uploadUserId"
          :type="uploaderPermissionTagType(detail)"
          effect="plain">{{ uploaderPermissionLabel(detail) }}</el-tag>
        <span v-else>{{ uploaderPermissionLabel(detail) }}</span>
      </el-descriptions-item>
      <el-descriptions-item label="所有者">
        {{ detail.ownerName || detail.ownerUserId || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="所属部门">
        {{ detail.deptName || detail.deptId || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="存储状态">
        {{ storageStatusLabel(detail.storageStatus) }}
      </el-descriptions-item>
      <el-descriptions-item label="ACL数量">
        {{ detail.aclEntryCount || 0 }}
      </el-descriptions-item>
      <el-descriptions-item label="业务引用">
        <el-button
          v-if="detail.referenceCount"
          type="text"
          @click="$emit('reference', detail)">{{ detail.referenceCount }} 项</el-button>
        <span v-else>0 项</span>
      </el-descriptions-item>
      <el-descriptions-item label="权限版本">
        {{
          detail.aclVersion !== undefined && detail.aclVersion !== null ? detail.aclVersion : '-'
        }}
      </el-descriptions-item>
      <el-descriptions-item label="文件大小">
        {{ formatFileSize(detail.fileSize) }}
      </el-descriptions-item>
      <el-descriptions-item label="内容类型">
        {{ detail.contentType || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="扩展名">
        {{ detail.extension || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="上传时间">
        {{ parseTime(detail.createTime) }}
      </el-descriptions-item>
      <el-descriptions-item label="过期时间">
        {{ parseTime(detail.expireTime) || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="移入回收站">
        {{ parseTime(detail.deletedTime) || '-' }}
      </el-descriptions-item>
      <el-descriptions-item label="存储相对路径" :span="2">
        {{ detail.storageKey }}
      </el-descriptions-item>
      <el-descriptions-item label="SHA-256" :span="2">
        <span class="file-hash">{{ detail.fileHash }}</span>
      </el-descriptions-item>
    </el-descriptions>
    <div class="dialog-footer" slot="footer">
      <el-button @click="dialogOpen = false">关 闭</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { accessTypeLabel, formatFileSize, statusLabel, storageStatusLabel } from './fileFormatters'

/**
 * 组件名称：FileDetailDialog
 * 功能说明：文件详情弹窗组件，展示文件的基础信息、权限与存储属性
 * 研发时间：2026/07/26
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} open   - 弹窗是否显示
 * @prop {Object}  detail - 文件详情数据
 *
 * 变量说明：
 *
 * 事件说明：
 * @event reference   - 查看文件业务引用
 * @event update:open - 更新弹窗显示状态
 *
 * 插槽说明：
 */

/**
 * 2026/07/26     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'FileDetailDialog',

  props:
  {
    open: { type: Boolean, default: false },
    detail: { type: Object, default: () => ({}) }
  },

  computed:
  {
    dialogOpen:
    {
      /**
       * 读取弹窗显示状态
       * @returns {Boolean} 弹窗是否显示
       */
      get()
      {
        return this.open
      },

      /**
       * 更新弹窗显示状态并向上抛出事件
       * @param {Boolean} value - 弹窗是否显示
       */
      set(value)
      {
        this.$emit('update:open', value)
      }
    }
  },

  methods:
  {
    /**
     * 访问类型文案
     */
    accessTypeLabel,

    /**
     * 文件大小格式化
     */
    formatFileSize,

    /**
     * 文件状态文案
     */
    statusLabel,

    /**
     * 存储状态文案
     */
    storageStatusLabel,

    /**
     * 上传人权限文案
     * @param {Object} file - 文件对象
     * @returns {String} 上传人权限文案
     */
    uploaderPermissionLabel(file)
    {
      if (file.accessType !== 'private')
      {
        return '公开文件无需单独授权'
      }
      if (!file.uploadUserId)
      {
        return '未记录上传人'
      }
      const accessEnabled =
        file.uploaderAccessEnabled === '1' || file.uploaderAccessEnabled === true
      if (file.uploadUserId === file.ownerUserId)
      {
        return accessEnabled ? '兼容访问已保留；同时为所有者' : '兼容访问已移除；仍通过所有者访问'
      }
      return accessEnabled ? '已保留，显式拒绝可覆盖' : '已移除，不再因上传身份放行'
    },

    /**
     * 上传人权限标签类型
     * @param {Object} file - 文件对象
     * @returns {String} 标签类型
     */
    uploaderPermissionTagType(file)
    {
      if (file.uploaderAccessEnabled !== '1' && file.uploaderAccessEnabled !== true)
      {
        return 'info'
      }
      return file.uploadUserId === file.ownerUserId ? 'primary' : 'warning'
    }
  }
}
</script>