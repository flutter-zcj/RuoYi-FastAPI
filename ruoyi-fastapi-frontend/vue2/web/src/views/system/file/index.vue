<style lang="scss" scoped>
</style>

<template>
  <div class="app-container">
    <file-search-form
      :query-params="queryParams"
      :date-range.sync="dateRange"
      :show="showSearch"
      :dept-options="fileDeptOptions"
      @query="handleQuery"
      @reset="resetQuery" />

    <file-statistics :stats="fileStats" />

    <el-row class="mb8" :gutter="10">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          size="mini"
          icon="el-icon-lock"
          :disabled="privateMultiple"
          v-hasPermi="['system:file:edit']"
          @click="handleAcl()">授权</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          size="mini"
          icon="el-icon-sort"
          :disabled="activeMultiple"
          v-hasPermi="['system:file:transfer']"
          @click="handleTransfer()">转移</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          size="mini"
          icon="el-icon-refresh-left"
          :disabled="restoreMultiple"
          v-hasPermi="['system:file:restore']"
          @click="handleRestore()">恢复</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          size="mini"
          icon="el-icon-delete-solid"
          :disabled="purgeMultiple"
          v-hasPermi="['system:file:purge']"
          @click="handlePurge()">清理</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-tooltip
          :disabled="!selectedReferencedCount"
          content="所选文件存在业务引用，请先解除引用"
          placement="top">
          <span>
            <el-button
              type="danger"
              plain
              size="mini"
              icon="el-icon-delete"
              :disabled="deleteMultiple"
              v-hasPermi="['system:file:remove']"
              @click="handleDelete()">删除</el-button>
          </span>
        </el-tooltip>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          size="mini"
          icon="el-icon-timer"
          v-hasPermi="['system:file:edit']"
          @click="$refs.retentionPolicyDrawer.open()">策略</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          size="mini"
          icon="el-icon-connection"
          v-hasPermi="['system:file:reconcile']"
          @click="$refs.reconcileDrawer.open()">对账</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          size="mini"
          icon="el-icon-bell"
          v-hasPermi="['system:file:list']"
          @click="$refs.retentionReminderDrawer.open()">提醒</el-button>
      </el-col>
      <right-toolbar :show-search.sync="showSearch" @queryTable="getList" />
    </el-row>

    <file-table
      :file-list="fileList"
      :loading="loading"
      @selection-change="handleSelectionChange"
      @view="handleView"
      @download="handleDownload"
      @reference="handleReference"
      @acl="handleAcl"
      @transfer="handleTransfer"
      @audit="handleAudit"
      @delete="handleDelete"
      @restore="handleRestore"
      @purge="handlePurge" />

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList" />

    <file-detail-dialog
      :open.sync="detailOpen"
      :detail="detail"
      @reference="handleReference" />
    <file-reference-drawer ref="referenceDrawer" />
    <file-retention-policy-drawer ref="retentionPolicyDrawer" />
    <file-retention-reminder-drawer ref="retentionReminderDrawer" @refresh="getList" />
    <file-acl-drawer ref="aclDrawer" @refresh="getList" />
    <file-transfer-dialog ref="transferDialog" @refresh="getList" />
    <file-audit-drawer ref="auditDrawer" />
    <file-reconcile-drawer ref="reconcileDrawer" @refresh="getList" />
  </div>
</template>

<script>
import {
  delFile,
  getFile,
  getFileAclDeptTree,
  getFileStats,
  listFile,
  purgeFile,
  restoreFile,
} from '@/api/system/file'
import FileAclDrawer from './components/FileAclDrawer.vue'
import FileAuditDrawer from './components/FileAuditDrawer.vue'
import FileDetailDialog from './components/FileDetailDialog.vue'
import FileReferenceDrawer from './components/FileReferenceDrawer.vue'
import FileReconcileDrawer from './components/FileReconcileDrawer.vue'
import FileRetentionPolicyDrawer from './components/FileRetentionPolicyDrawer.vue'
import FileRetentionReminderDrawer from './components/FileRetentionReminderDrawer.vue'
import FileSearchForm from './components/FileSearchForm.vue'
import FileStatistics from './components/FileStatistics.vue'
import FileTable from './components/FileTable.vue'
import FileTransferDialog from './components/FileTransferDialog.vue'

/**
 * 组件名称：File
 * 功能说明：文件管理页面，支持文件查询、下载、授权、转移、恢复、清理与保留策略配置
 * 研发时间：2026/07/26
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/07/26     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'File',

  components:
  {
    FileAclDrawer,
    FileAuditDrawer,
    FileDetailDialog,
    FileReferenceDrawer,
    FileReconcileDrawer,
    FileRetentionPolicyDrawer,
    FileRetentionReminderDrawer,
    FileSearchForm,
    FileStatistics,
    FileTable,
    FileTransferDialog
  },

  data()
  {
    return {
      fileList: [],
      loading: true,
      showSearch: true,
      ids: [],
      privateIds: [],
      activeMultiple: true,
      deleteMultiple: true,
      privateMultiple: true,
      restoreMultiple: true,
      purgeMultiple: true,
      selectedReferencedCount: 0,
      total: 0,
      dateRange: [],
      detailOpen: false,
      detail: {},
      fileDeptOptions: [],
      fileStats: {
        totalCount: 0,
        totalSize: 0,
        publicSize: 0,
        privateSize: 0,
        activeCount: 0,
        deletedCount: 0,
        expiredCount: 0,
        retentionExpiringCount: 0,
        aclExpiringCount: 0,
      },
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        originalName: undefined,
        accessType: undefined,
        status: 'active',
        createBy: undefined,
        ownerName: undefined,
        deptId: undefined,
        expirationStatus: undefined,
      },
    }
  },

  created()
  {
    this.getList()
    getFileAclDeptTree().then((response) =>
    {
      this.fileDeptOptions = response.data
    })
  },

  methods:
  {
    /**
     * 查询文件列表
     */
    getList()
    {
      this.loading = true
      const query = this.addDateRange({ ...this.queryParams }, this.dateRange)
      listFile(query)
        .then((response) =>
        {
          this.fileList = response.rows
          this.total = response.total
        })
        .finally(() =>
        {
          this.loading = false
        })
      getFileStats(query).then((response) =>
      {
        Object.assign(this.fileStats, response.data)
      })
    },

    /**
     * 搜索按钮操作
     */
    handleQuery()
    {
      this.queryParams.pageNum = 1
      this.getList()
    },

    /**
     * 重置按钮操作
     */
    resetQuery()
    {
      this.queryParams.pageNum = 1
      this.getList()
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的行数据
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.fileId)
      this.privateIds = selection
        .filter((item) => item.status === 'active' && item.accessType === 'private')
        .map((item) => item.fileId)
      this.selectedReferencedCount = selection.filter((item) => item.referenceCount > 0).length
      this.activeMultiple = !selection.length || selection.some((item) => item.status !== 'active')
      this.deleteMultiple = this.activeMultiple || this.selectedReferencedCount > 0
      this.privateMultiple = !this.privateIds.length
      this.restoreMultiple =
        !selection.length || selection.some((item) => item.status !== 'deleted')
      this.purgeMultiple =
        !selection.length || selection.some((item) => !['deleted', 'purging'].includes(item.status))
    },

    /**
     * 查看文件详情
     * @param {Object} row - 当前行数据
     */
    handleView(row)
    {
      getFile(row.fileId).then((response) =>
      {
        this.detail = response.data
        this.detailOpen = true
      })
    },

    /**
     * 下载文件
     * @param {Object} row - 当前行数据
     */
    handleDownload(row)
    {
      const displayName = encodeURIComponent(row.storedName || 'file')
      this.$download.file(`/system/file/download/${row.fileId}/${displayName}`)
    },

    /**
     * 查看文件业务引用
     * @param {Object} row - 当前行数据
     */
    handleReference(row)
    {
      this.$refs.referenceDrawer.open(row)
    },

    /**
     * 配置文件访问权限
     * @param {Object} row - 当前行数据
     */
    handleAcl(row)
    {
      this.$refs.aclDrawer.open(row, this.ids, this.privateIds)
    },

    /**
     * 转移文件
     * @param {Object} row - 当前行数据
     */
    handleTransfer(row)
    {
      this.$refs.transferDialog.open(row, this.ids)
    },

    /**
     * 查看文件访问审计
     * @param {Object} row - 当前行数据
     */
    handleAudit(row)
    {
      this.$refs.auditDrawer.open(row)
    },

    /**
     * 删除文件
     * @param {Object} row - 当前行数据
     */
    handleDelete(row)
    {
      const isSingle = row && row.fileId
      if (isSingle && row.referenceCount > 0)
      {
        this.$modal.msgWarning('文件仍被业务引用，请先解除引用后再删除')
        return
      }
      if (!isSingle && this.selectedReferencedCount > 0)
      {
        this.$modal.msgWarning('所选文件存在业务引用，请先解除引用后再删除')
        return
      }
      const fileIds = isSingle ? row.fileId : this.ids
      const fileName = isSingle ? row.originalName : `${this.ids.length}个文件`
      this.$modal
        .confirm(`是否确认将文件“${fileName}”移入回收站?`)
        .then(() => delFile(fileIds))
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('文件已移入回收站')
        })
        .catch(() => {})
    },

    /**
     * 恢复文件
     * @param {Object} row - 当前行数据
     */
    handleRestore(row)
    {
      const isSingle = row && row.fileId
      const fileIds = isSingle ? row.fileId : this.ids
      const fileName = isSingle ? row.originalName : `${this.ids.length}个文件`
      this.$modal
        .confirm(`是否确认恢复文件“${fileName}”?`)
        .then(() => restoreFile(fileIds))
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('文件恢复成功')
        })
        .catch(() => {})
    },

    /**
     * 永久清理文件
     * @param {Object} row - 当前行数据
     */
    handlePurge(row)
    {
      const isSingle = row && row.fileId
      const fileIds = isSingle ? row.fileId : this.ids.join(',')
      const fileName = isSingle ? row.originalName : `${this.ids.length}个文件`
      this.$modal
        .confirm(`永久清理后无法恢复，是否确认清理文件“${fileName}”?`)
        .then(() => purgeFile(fileIds))
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('文件已永久清理')
        })
        .catch(() => {})
    }
  }
}
</script>