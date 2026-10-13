<style lang="scss" scoped>
.drawer-content { padding: 0 20px 20px; }
.file-action-danger { color: #f56c6c; }
</style>

<template>
  <div>
    <el-drawer
      title="文件保留策略"
      :visible.sync="visible"
      size="900px"
      append-to-body>
      <div class="drawer-content">
        <el-alert
          class="mb8"
          title="策略仅应用于后续新建或重建的业务引用；未配置或已停用的业务类型永久保留，既有引用不会被追溯修改。"
          type="info"
          :closable="false"
          show-icon />
        <el-row class="mb8" :gutter="10">
          <el-col :span="1.5">
            <el-button
              type="primary"
              plain
              size="mini"
              icon="el-icon-plus"
              @click="handleAdd">新增</el-button>
          </el-col>
        </el-row>
        <el-table v-loading="loading" :data="policyList">
          <el-table-column
            label="业务类型"
            align="center"
            prop="businessType"
            min-width="140"
            :show-overflow-tooltip="true" />
          <el-table-column
            label="保留天数"
            align="center"
            prop="retentionDays"
            width="100" />
          <el-table-column
            label="状态"
            align="center"
            prop="status"
            width="90">
            <template slot-scope="scope">
              <el-tag :type="scope.row.status === '0' ? 'success' : 'info'">
                {{ scope.row.status === '0' ? '启用' : '停用' }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column
            label="备注"
            align="left"
            prop="remark"
            min-width="160"
            :show-overflow-tooltip="true">
            <template slot-scope="scope">{{ scope.row.remark || '-' }}</template>
          </el-table-column>
          <el-table-column
            label="更新时间"
            align="center"
            prop="updateTime"
            width="180">
            <template slot-scope="scope">
              {{ parseTime(scope.row.updateTime) || '-' }}
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            width="90"
            fixed="right">
            <template slot-scope="scope">
              <el-tooltip content="修改" placement="top">
                <el-button
                  type="text"
                  icon="el-icon-edit"
                  @click="handleEdit(scope.row)" />
              </el-tooltip>
              <el-tooltip content="删除" placement="top">
                <el-button
                  class="file-action-danger"
                  type="text"
                  icon="el-icon-delete"
                  @click="handleDelete(scope.row)" />
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-drawer>

    <el-dialog
      :title="formTitle"
      :visible.sync="formOpen"
      width="520px"
      append-to-body>
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        label-width="90px">
        <el-form-item label="业务类型" prop="businessType">
          <el-input
            v-model="form.businessType"
            :disabled="editing"
            maxlength="50"
            placeholder="请输入业务类型" />
        </el-form-item>
        <el-form-item label="保留天数" prop="retentionDays">
          <el-input-number
            v-model="form.retentionDays"
            controls-position="right"
            :min="1"
            :max="36500"
            :precision="0"
            style="width: 100%" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio label="0">启用</el-radio>
            <el-radio label="1">停用</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            :rows="3"
            maxlength="500"
            show-word-limit
            placeholder="请输入备注" />
        </el-form-item>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <el-button
          type="primary"
          :loading="saving"
          @click="submit">
          确 定
        </el-button>
        <el-button @click="formOpen = false">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  addFileRetentionPolicy,
  delFileRetentionPolicy,
  listFileRetentionPolicy,
  updateFileRetentionPolicy,
} from '@/api/system/file'

/**
 * 组件名称：FileRetentionPolicyDrawer
 * 功能说明：文件保留策略抽屉组件，用于查询、新增、修改与删除业务类型的文件保留策略
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
  name: 'FileRetentionPolicyDrawer',

  data()
  {
    return {
      visible: false,
      loading: false,
      policyList: [],
      formOpen: false,
      editing: false,
      saving: false,
      form: {
        businessType: undefined,
        retentionDays: 30,
        status: '0',
        remark: undefined,
      },
      rules: {
        businessType: [{ required: true, message: '请输入业务类型', trigger: 'blur' }],
        retentionDays: [{ required: true, message: '请输入保留天数', trigger: 'blur' }],
      },
    }
  },

  computed:
  {
    /**
     * 表单弹窗标题
     * @returns {String} 弹窗标题
     */
    formTitle()
    {
      return this.editing ? '修改文件保留策略' : '新增文件保留策略'
    }
  },

  methods:
  {
    /**
     * 打开抽屉
     */
    open()
    {
      this.visible = true
      this.getList()
    },

    /**
     * 查询保留策略列表
     */
    getList()
    {
      this.loading = true
      listFileRetentionPolicy()
        .then((response) =>
        {
          this.policyList = response.data
        })
        .finally(() =>
        {
          this.loading = false
        })
    },

    /**
     * 重置表单
     */
    resetForm()
    {
      Object.assign(this.form, {
        businessType: undefined,
        retentionDays: 30,
        status: '0',
        remark: undefined,
      })
      this.$nextTick(() => this.$refs.form.clearValidate())
    },

    /**
     * 新增按钮操作
     */
    handleAdd()
    {
      this.editing = false
      this.resetForm()
      this.formOpen = true
    },

    /**
     * 修改按钮操作
     * @param {Object} row - 当前行数据
     */
    handleEdit(row)
    {
      this.editing = true
      Object.assign(this.form, {
        businessType: row.businessType,
        retentionDays: row.retentionDays,
        status: row.status,
        remark: row.remark,
      })
      this.formOpen = true
      this.$nextTick(() => this.$refs.form.clearValidate())
    },

    /**
     * 提交表单
     */
    submit()
    {
      this.$refs.form.validate((valid) =>
      {
        if (!valid)
        {
          return
        }
        this.saving = true
        const submitRequest = this.editing
          ? updateFileRetentionPolicy(this.form)
          : addFileRetentionPolicy(this.form)
        submitRequest
          .then(() =>
          {
            this.$modal.msgSuccess(this.editing ? '修改成功' : '新增成功')
            this.formOpen = false
            this.getList()
          })
          .finally(() =>
          {
            this.saving = false
          })
      })
    },

    /**
     * 删除按钮操作
     * @param {Object} row - 当前行数据
     */
    handleDelete(row)
    {
      this.$modal
        .confirm(`是否确认删除业务类型“${row.businessType}”的保留策略?`)
        .then(() => delFileRetentionPolicy(row.businessType))
        .then(() =>
        {
          this.$modal.msgSuccess('删除成功')
          this.getList()
        })
        .catch(() => {})
    }
  }
}
</script>