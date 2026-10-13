<style lang="scss" scoped>
.drawer-content { padding: 0 20px 20px; }
.drawer-footer { padding-top: 20px; text-align: right; }
.permission-section { margin-bottom: 20px; }
.permission-section__header { display: flex; align-items: center; justify-content: space-between; margin: 18px 0 8px; }
.permission-section__title { font-size: 15px; font-weight: 600; color: #303133; }
.file-action-danger { color: #f56c6c; }
</style>

<template>
  <el-drawer
    :title="drawerTitle"
    :visible.sync="visible"
    size="70%"
    append-to-body
    :wrapper-closable="!saving">
    <div class="drawer-content">
      <el-alert
        class="mb8"
        :title="
          batchMode
            ? '批量操作仅覆盖所选文件的显式授权；内置权限仍按各文件规则生效。'
            : '内置权限只读展示；管理员和所有者不可被拒绝，启用的上传人权限可被匹配的显式拒绝覆盖。'
        "
        type="info"
        :closable="false"
        show-icon/>
      <div class="permission-section" v-if="!batchMode">
        <div class="permission-section__title">内置权限</div>
        <el-table
          v-loading="loading"
          :data="builtinPermissions"
          border>
          <el-table-column
            label="权限来源"
            align="center"
            width="110">
            <template slot-scope="scope">
              <el-tag :type="builtinSourceType(scope.row.source)" effect="plain">{{ builtinSourceLabel(scope.row.source) }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="权限主体"
            align="center"
            min-width="180">
            <template slot-scope="scope">
              {{ scope.row.subjectName || scope.row.subjectId || '-' }}
            </template>
          </el-table-column>
          <el-table-column
            label="规则"
            align="center"
            width="110">
            <template slot-scope="scope">
              <el-tag :type="scope.row.enabled ? 'success' : 'info'" effect="plain">{{ scope.row.enabled ? '允许下载' : '已移除' }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="拒绝覆盖"
            align="center"
            width="135">
            <template slot-scope="scope">
              <el-tag
                v-if="scope.row.enabled"
                :type="scope.row.denyOverridable ? 'warning' : 'info'"
                effect="plain">{{ scope.row.denyOverridable ? '可以覆盖' : '不可覆盖' }}</el-tag>
              <span v-else>-</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="description"
            label="权限说明"
            align="left"
            min-width="320"
            show-overflow-tooltip/>
        </el-table>
      </div>
      <div class="permission-section__header">
        <span class="permission-section__title">显式授权</span>
        <el-button
          type="primary"
          plain
          size="mini"
          icon="el-icon-plus"
          @click="addEntry">添加授权</el-button>
      </div>
      <el-table v-loading="loading" :data="entries">
        <el-table-column
          label="主体类型"
          align="center"
          width="125">
          <template slot-scope="scope">
            <el-select v-model="scope.row.subjectType" @change="handleSubjectTypeChange(scope.row)">
              <el-option label="用户" value="user"/>
              <el-option label="角色" value="role"/>
              <el-option label="部门" value="dept"/>
            </el-select>
          </template>
        </el-table-column>
        <el-table-column
          label="授权主体"
          align="center"
          min-width="230">
          <template slot-scope="scope">
            <treeselect
              v-if="scope.row.subjectType === 'dept'"
              v-model="scope.row.subjectId"
              :options="deptOptions"
              :normalizer="normalizer"
              :append-to-body="true"
              :z-index="3000"
              placeholder="请选择部门"
              :clearable="true"
              :searchable="true"
              style="width: 100%"/>
            <el-select
              v-else
              v-model="scope.row.subjectId"
              filterable
              remote
              clearable
              :loading="scope.row.subjectLoading"
              :remote-method="getSubjectRemoteMethod(scope.row)"
              placeholder="输入名称搜索"
              style="width: 100%"
              @visible-change="handleSubjectVisible(scope.row, $event)">
              <el-option
                v-for="item in scope.row.subjectOptions"
                :key="item.subjectId"
                :label="item.subjectName"
                :value="item.subjectId"/>
            </el-select>
          </template>
        </el-table-column>
        <el-table-column
          label="规则"
          align="center"
          width="120">
          <template slot-scope="scope">
            <el-select v-model="scope.row.effect">
              <el-option label="允许下载" value="allow"/>
              <el-option label="拒绝下载" value="deny"/>
            </el-select>
          </template>
        </el-table-column>
        <el-table-column
          label="包含下级部门"
          align="center"
          width="125">
          <template slot-scope="scope">
            <el-switch v-if="scope.row.subjectType === 'dept'" v-model="scope.row.includeChildren"/>
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column
          label="有效期至"
          align="center"
          width="310">
          <template slot-scope="scope">
            <BusinessDateTimePicker v-model="scope.row.expireTime" label="有效期至，留空为永久有效"/>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          align="center"
          width="70">
          <template slot-scope="scope">
            <el-tooltip content="删除" placement="top">
              <el-button
                class="file-action-danger"
                type="text"
                icon="el-icon-delete"
                @click="removeEntry(scope.$index)"/>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <div class="drawer-footer">
        <el-button
          type="primary"
          :loading="saving"
          @click="submit">保 存</el-button>
        <el-button @click="visible = false">取 消</el-button>
      </div>
    </div>
  </el-drawer>
</template>

<script>
/**
 * 组件名称：FileAclDrawer
 * 功能说明：文件访问权限抽屉，支持单文件与批量文件的显式授权配置
 * 研发时间：2026/07/26
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} visible            - 抽屉是否可见
 * @data {Boolean} loading            - 数据加载状态
 * @data {Boolean} saving             - 保存提交中状态
 * @data {String}  fileId             - 当前文件标识
 * @data {String}  fileName           - 当前文件名
 * @data {String}  fileIds            - 批量文件标识集合
 * @data {Boolean} batchMode          - 是否为批量授权模式
 * @data {Number}  batchCount         - 批量授权文件数量
 * @data {Number}  aclVersion         - 权限版本号
 * @data {Array}   builtinPermissions - 内置权限列表
 * @data {Array}   entries            - 显式授权列表
 * @data {Array}   deptOptions        - 部门树选项
 *
 * 事件说明：
 * @event normalizer              - 部门树数据标准化
 * @event open                    - 打开抽屉并加载权限数据
 * @event builtinSourceLabel      - 内置权限来源文本转换
 * @event builtinSourceType       - 内置权限来源标签类型转换
 * @event addEntry                - 新增显式授权项
 * @event removeEntry             - 删除显式授权项
 * @event handleSubjectTypeChange - 授权主体类型变更回调
 * @event handleSubjectVisible    - 授权主体下拉可见性变更回调
 * @event getSubjectRemoteMethod  - 生成授权主体远程搜索方法
 * @event searchSubjectOptions    - 搜索授权主体选项
 * @event submit                  - 提交权限保存
 *
 * 插槽说明：
 */

/**
 * 2026/07/26     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { prepareTimeFields, serializeTimeFieldsForSubmit } from '@/utils/time'
import BusinessDateTimePicker from '@/components/BusinessDateTimePicker'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import {
  batchSaveFileAcl,
  getFileAclDeptTree,
  listFileAcl,
  saveFileAcl,
  searchFileAclSubjects,
} from '@/api/system/file'

export default
{
  name: 'FileAclDrawer',

  components:
  {
    Treeselect,
    BusinessDateTimePicker
  },

  data()
  {
    return {
      visible: false,
      loading: false,
      saving: false,
      fileId: '',
      fileName: '',
      fileIds: '',
      batchMode: false,
      batchCount: 0,
      aclVersion: 0,
      builtinPermissions: [],
      entries: [],
      deptOptions: [],
    }
  },

  computed:
  {
    /**
     * 抽屉标题，依据模式展示批量或单文件标题
     * @returns {String} 抽屉标题
     */
    drawerTitle()
    {
      return this.batchMode ? `批量授权 - ${this.batchCount}个文件` : `访问权限 - ${this.fileName}`
    }
  },

  methods:
  {
    /**
     * 部门树数据标准化
     * @param {Object} node - 部门树节点
     * @returns {Object} 标准化后的节点
     */
    normalizer(node)
    {
      return {
        id: node.id,
        label: node.label,
        children: node.children,
      }
    },

    /**
     * 打开抽屉并加载权限数据
     * @param {Object} row - 当前文件行数据
     * @param {Array} selectedIds - 已选中的全部文件标识
     * @param {Array} selectedPrivateIds - 已选中的受保护文件标识
     */
    open(row, selectedIds, selectedPrivateIds)
    {
      this.batchMode = !(row && row.fileId)
      if (this.batchMode && !selectedPrivateIds.length) {
        this.$modal.msgWarning('请选择正常状态的受保护文件')
        return
      }
      this.fileId = row && row.fileId ? row.fileId : ''
      this.fileIds = this.batchMode ? selectedPrivateIds.join(',') : row.fileId
      this.batchCount = this.batchMode ? selectedPrivateIds.length : 0
      this.fileName = row && row.originalName ? row.originalName : ''
      if (this.batchMode && selectedPrivateIds.length < selectedIds.length) {
        this.$modal.msgWarning(
          `已忽略${selectedIds.length - selectedPrivateIds.length}个公开或非正常状态的文件`
        )
      }
      this.visible = true
      this.loading = true
      this.builtinPermissions = []
      if (this.batchMode) {
        this.aclVersion = 0
        this.entries = []
        getFileAclDeptTree()
          .then((response) => {
            this.deptOptions = response.data
          })
          .finally(() => {
            this.loading = false
          })
        return
      }
      Promise.all([listFileAcl(row.fileId), getFileAclDeptTree()])
        .then(([aclResponse, deptResponse]) => {
          this.deptOptions = deptResponse.data
          this.aclVersion = aclResponse.data.aclVersion
          this.builtinPermissions = aclResponse.data.builtinPermissions || []
          this.entries = aclResponse.data.entries.map((item) =>
            prepareTimeFields(
              {
                subjectType: item.subjectType,
                subjectId: item.subjectId,
                effect: item.effect,
                includeChildren: item.includeChildren,
                expireTime: item.expireTime,
                subjectLoading: false,
                subjectOptions: [
                  {
                    subjectId: item.subjectId,
                    subjectName: item.subjectName,
                  },
                ],
              },
              ['expireTime']
            )
          )
        })
        .finally(() => {
          this.loading = false
        })
    },

    /**
     * 内置权限来源文本转换
     * @param {String} source - 权限来源
     * @returns {String} 权限来源文本
     */
    builtinSourceLabel(source)
    {
      return (
        {
          admin: '平台管理员',
          owner: '文件所有者',
          uploader: '文件上传人',
        }[source] || source
      )
    },

    /**
     * 内置权限来源标签类型转换
     * @param {String} source - 权限来源
     * @returns {String} 标签类型
     */
    builtinSourceType(source)
    {
      return (
        {
          admin: 'danger',
          owner: 'primary',
          uploader: 'warning',
        }[source] || 'info'
      )
    },

    /**
     * 新增显式授权项
     */
    addEntry()
    {
      this.entries.push({
        subjectType: 'user',
        subjectId: undefined,
        effect: 'allow',
        includeChildren: false,
        expireTime: undefined,
        subjectLoading: false,
        subjectOptions: [],
      })
    },

    /**
     * 删除显式授权项
     * @param {Number} index - 授权项索引
     */
    removeEntry(index)
    {
      this.entries.splice(index, 1)
    },

    /**
     * 授权主体类型变更回调
     * @param {Object} row - 当前授权项
     */
    handleSubjectTypeChange(row)
    {
      row.subjectId = undefined
      row.includeChildren = false
      row.subjectOptions = []
    },

    /**
     * 授权主体下拉可见性变更回调
     * @param {Object} row - 当前授权项
     * @param {Boolean} visible - 下拉是否可见
     */
    handleSubjectVisible(row, visible)
    {
      if (visible && !row.subjectOptions.length) {
        this.searchSubjectOptions(row, '')
      }
    },

    /**
     * 生成授权主体远程搜索方法
     * @param {Object} row - 当前授权项
     * @returns {Function} 远程搜索方法
     */
    getSubjectRemoteMethod(row)
    {
      return (keyword) => this.searchSubjectOptions(row, keyword)
    },

    /**
     * 搜索授权主体选项
     * @param {Object} row - 当前授权项
     * @param {String} keyword - 搜索关键字
     */
    searchSubjectOptions(row, keyword)
    {
      const subjectType = row.subjectType
      row.subjectLoading = true
      searchFileAclSubjects({ subjectType, keyword })
        .then((response) => {
          if (row.subjectType === subjectType) {
            row.subjectOptions = response.data
          }
          row.subjectLoading = false
        })
        .catch(() => {
          row.subjectLoading = false
        })
    },

    /**
     * 提交权限保存
     */
    async submit()
    {
      if (this.entries.some((item) => !item.subjectId)) {
        this.$modal.msgError('请选择完整的授权主体')
        return
      }
      const convertedEntries = []
      for (const item of this.entries) {
        const converted = await serializeTimeFieldsForSubmit(item, ['expireTime'])
        if (!converted) return
        convertedEntries.push(converted)
      }
      const aclEntries = convertedEntries.map((item) => ({
        subjectType: item.subjectType,
        subjectId: item.subjectId,
        effect: item.effect,
        includeChildren: item.subjectType === 'dept' && item.includeChildren,
        expireTime: item.expireTime,
      }))
      const saveAcl = () => {
        this.saving = true
        const saveRequest = this.batchMode
          ? batchSaveFileAcl({
              fileIds: this.fileIds,
              entries: aclEntries,
            })
          : saveFileAcl(this.fileId, {
              aclVersion: this.aclVersion,
              entries: aclEntries,
            })
        saveRequest
          .then(() => {
            this.$modal.msgSuccess(this.batchMode ? '文件权限批量保存成功' : '文件权限保存成功')
            this.visible = false
            this.$emit('refresh')
          })
          .finally(() => {
            this.saving = false
          })
      }
      if (this.batchMode) {
        this.$modal
          .confirm(`将覆盖所选${this.batchCount}个文件的全部授权，是否继续?`)
          .then(saveAcl)
          .catch(() => {})
      } else {
        saveAcl()
      }
    },
  },
}
</script>