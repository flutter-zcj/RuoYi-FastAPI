<style lang="scss" scoped>
.file-action-disabled-wrapper { display: inline-flex; align-items: center; margin-left: 10px; vertical-align: middle; }
.file-action-danger { color: #f56c6c; }
.file-action-success { color: #67c23a; }
</style>

<template>
  <el-table
    v-loading="loading"
    :data="fileList"
    @selection-change="$emit('selection-change', $event)">
    <el-table-column
      type="selection"
      width="50"
      align="center" />
    <el-table-column
      label="原始文件名"
      align="left"
      prop="originalName"
      min-width="200"
      :show-overflow-tooltip="true" />
    <el-table-column
      label="访问类型"
      align="center"
      prop="accessType"
      width="110">
      <template slot-scope="scope">
        <el-tag v-if="scope.row.accessType === 'public'" type="success">公开</el-tag>
        <el-tag v-else type="warning">受保护</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="扩展名"
      align="center"
      prop="extension"
      width="90" />
    <el-table-column
      label="文件大小"
      align="right"
      prop="fileSize"
      width="120">
      <template slot-scope="scope">{{ formatFileSize(scope.row.fileSize) }}</template>
    </el-table-column>
    <el-table-column
      label="上传用户"
      align="center"
      prop="createBy"
      width="120"
      :show-overflow-tooltip="true" />
    <el-table-column
      label="所有者"
      align="center"
      prop="ownerName"
      width="120"
      :show-overflow-tooltip="true">
      <template slot-scope="scope">
        {{ scope.row.ownerName || scope.row.ownerUserId || '-' }}
      </template>
    </el-table-column>
    <el-table-column
      label="所属部门"
      align="center"
      prop="deptName"
      width="140"
      :show-overflow-tooltip="true">
      <template slot-scope="scope">
        {{ scope.row.deptName || scope.row.deptId || '-' }}
      </template>
    </el-table-column>
    <el-table-column
      label="文件有效期"
      align="center"
      prop="expireTime"
      width="150">
      <template slot-scope="scope">
        <el-tag :type="expirationTagType(scope.row.expireTime)" effect="plain">{{ expirationLabel(scope.row.expireTime) }}</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="ACL"
      align="center"
      width="130">
      <template slot-scope="scope">
        <el-tooltip
          v-if="scope.row.aclNearestExpireTime"
          :content="`最近过期：${parseTime(scope.row.aclNearestExpireTime)}`"
          placement="top">
          <el-tag :type="isAclExpiring(scope.row.aclNearestExpireTime) ? 'warning' : 'info'" effect="plain">
            {{ scope.row.aclEntryCount }} 项
            <span v-if="isAclExpiring(scope.row.aclNearestExpireTime)">· 即将过期</span>
          </el-tag>
        </el-tooltip>
        <span v-else>{{ scope.row.aclEntryCount || 0 }} 项</span>
      </template>
    </el-table-column>
    <el-table-column
      label="业务引用"
      align="center"
      prop="referenceCount"
      width="100">
      <template slot-scope="scope">
        <el-button
          v-if="scope.row.referenceCount"
          type="text"
          @click="$emit('reference', scope.row)">{{ scope.row.referenceCount }} 项</el-button>
        <span v-else>0 项</span>
      </template>
    </el-table-column>
    <el-table-column
      label="存储状态"
      align="center"
      prop="storageStatus"
      width="110">
      <template slot-scope="scope">
        <el-tag :type="storageStatusTagType(scope.row.storageStatus)" effect="plain">{{ storageStatusLabel(scope.row.storageStatus) }}</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="上传时间"
      align="center"
      prop="createTime"
      width="180">
      <template slot-scope="scope">{{ parseTime(scope.row.createTime) }}</template>
    </el-table-column>
    <el-table-column
      label="状态"
      align="center"
      prop="status"
      width="90">
      <template slot-scope="scope">
        <el-tag v-if="scope.row.status === 'active'" type="success">正常</el-tag>
        <el-tag v-else-if="scope.row.status === 'deleted'" type="info">已删除</el-tag>
        <el-tag v-else type="warning">清理中</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="操作"
      align="center"
      class-name="small-padding fixed-width"
      width="250"
      fixed="right">
      <template slot-scope="scope">
        <el-tooltip content="详细" placement="top">
          <el-button
            type="text"
            icon="el-icon-view"
            v-hasPermi="['system:file:query']"
            @click="$emit('view', scope.row)" />
        </el-tooltip>
        <el-tooltip v-if="scope.row.status === 'active'" content="下载" placement="top">
          <el-button
            type="text"
            icon="el-icon-download"
            v-hasPermi="['system:file:download']"
            @click="$emit('download', scope.row)" />
        </el-tooltip>
        <el-tooltip
          v-if="scope.row.accessType === 'private' && scope.row.status === 'active'"
          content="授权"
          placement="top">
          <el-button
            type="text"
            icon="el-icon-lock"
            v-hasPermi="['system:file:edit']"
            @click="$emit('acl', scope.row)" />
        </el-tooltip>
        <el-tooltip v-if="scope.row.status === 'active'" content="转移" placement="top">
          <el-button
            type="text"
            icon="el-icon-sort"
            v-hasPermi="['system:file:transfer']"
            @click="$emit('transfer', scope.row)" />
        </el-tooltip>
        <el-tooltip content="审计" placement="top">
          <el-button
            type="text"
            icon="el-icon-tickets"
            v-hasPermi="['system:file:query']"
            @click="$emit('audit', scope.row)" />
        </el-tooltip>
        <el-tooltip
          v-if="scope.row.status === 'active' && scope.row.referenceCount"
          content="存在业务引用，无法删除"
          placement="top">
          <span class="file-action-disabled-wrapper">
            <el-button
              class="file-action-danger"
              type="text"
              icon="el-icon-delete"
              disabled
              v-hasPermi="['system:file:remove']" />
          </span>
        </el-tooltip>
        <el-tooltip v-else-if="scope.row.status === 'active'" content="删除" placement="top">
          <el-button
            class="file-action-danger"
            type="text"
            icon="el-icon-delete"
            v-hasPermi="['system:file:remove']"
            @click="$emit('delete', scope.row)" />
        </el-tooltip>
        <el-tooltip v-if="scope.row.status === 'deleted'" content="恢复" placement="top">
          <el-button
            class="file-action-success"
            type="text"
            icon="el-icon-refresh-left"
            v-hasPermi="['system:file:restore']"
            @click="$emit('restore', scope.row)" />
        </el-tooltip>
        <el-tooltip
          v-if="scope.row.status !== 'active'"
          :content="scope.row.status === 'purging' ? '重试永久清理' : '永久清理'"
          placement="top">
          <el-button
            class="file-action-danger"
            type="text"
            icon="el-icon-delete-solid"
            v-hasPermi="['system:file:purge']"
            @click="$emit('purge', scope.row)" />
        </el-tooltip>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import {
  expirationLabel,
  expirationTagType,
  formatFileSize,
  isAclExpiring,
  storageStatusLabel,
  storageStatusTagType,
} from './fileFormatters'

/**
 * 组件名称：FileTable
 * 功能说明：文件列表表格组件，展示文件信息并触发查看、下载、授权、转移、删除等操作
 * 研发时间：2026/07/26
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Array}   fileList - 文件列表数据
 * @prop {Boolean} loading  - 是否显示加载状态
 *
 * 变量说明：
 *
 * 事件说明：
 * @event selection-change - 表格选中项变化
 * @event view             - 查看文件详情
 * @event download         - 下载文件
 * @event reference        - 查看文件业务引用
 * @event acl              - 配置文件访问权限
 * @event transfer         - 转移文件
 * @event audit            - 查看文件访问审计
 * @event delete           - 删除文件
 * @event restore          - 恢复文件
 * @event purge            - 永久清理文件
 *
 * 插槽说明：
 */

/**
 * 2026/07/26     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'FileTable',

  props:
  {
    fileList: { type: Array, default: () => [] },
    loading: { type: Boolean, default: false }
  },

  methods:
  {
    /**
     * 文件有效期文案
     */
    expirationLabel,

    /**
     * 文件有效期标签类型
     */
    expirationTagType,

    /**
     * 文件大小格式化
     */
    formatFileSize,

    /**
     * 判断 ACL 是否即将过期
     */
    isAclExpiring,

    /**
     * 存储状态文案
     */
    storageStatusLabel,

    /**
     * 存储状态标签类型
     */
    storageStatusTagType
  }
}
</script>