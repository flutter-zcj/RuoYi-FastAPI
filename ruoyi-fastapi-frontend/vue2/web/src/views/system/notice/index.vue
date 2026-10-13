<style lang="scss" scoped></style>

<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      v-show="showSearch"
      label-width="68px">
      <el-form-item label="公告标题" prop="noticeTitle">
        <el-input
          v-model="queryParams.noticeTitle"
          placeholder="请输入公告标题"
          clearable
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="操作人员" prop="createBy">
        <el-input
          v-model="queryParams.createBy"
          placeholder="请输入操作人员"
          clearable
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="类型" prop="noticeType">
        <el-select
          v-model="queryParams.noticeType"
          placeholder="公告类型"
          clearable>
          <el-option
            v-for="dict in dict.type.sys_notice_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery">搜索</el-button>
        <el-button
          icon="el-icon-refresh"
          size="mini"
          @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row class="mb8" :gutter="10">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          v-hasPermi="['system:notice:add']"
          @click="handleAdd">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          v-hasPermi="['system:notice:edit']"
          @click="handleUpdate">修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          v-hasPermi="['system:notice:remove']"
          @click="handleDelete">删除</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="noticeList"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55"
        align="center"/>
      <el-table-column
        label="序号"
        align="center"
        prop="noticeId"
        width="100"/>
      <el-table-column
        label="公告标题"
        align="center"
        :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <a
            class="link-type"
            style="cursor: pointer"
            @click="handleViewData(scope.row)">{{ scope.row.noticeTitle }}</a>
        </template>
      </el-table-column>
      <el-table-column
        label="公告类型"
        align="center"
        prop="noticeType"
        width="100">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_notice_type" :value="scope.row.noticeType"/>
        </template>
      </el-table-column>
      <el-table-column
        label="状态"
        align="center"
        prop="status"
        width="100">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_notice_status" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column
        label="创建者"
        align="center"
        prop="createBy"
        width="100"/>
      <el-table-column
        label="创建时间"
        align="center"
        prop="createTime"
        width="100">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-user"
            v-hasPermi="['system:notice:list']"
            @click="handleReadUsers(scope.row)">阅读用户</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            v-hasPermi="['system:notice:edit']"
            @click="handleUpdate(scope.row)">修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            v-hasPermi="['system:notice:remove']"
            @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"/>

    <!-- 添加或修改公告对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="open"
      width="780px"
      append-to-body>
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        label-width="80px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="公告标题" prop="noticeTitle">
              <el-input
                v-model="form.noticeTitle"
                placeholder="请输入公告标题"/>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="公告类型" prop="noticeType">
              <el-select v-model="form.noticeType" placeholder="请选择公告类型">
                <el-option
                  v-for="dict in dict.type.sys_notice_type"
                  :key="dict.value"
                  :label="dict.label"
                  :value="dict.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="状态">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in dict.type.sys_notice_status"
                  :key="dict.value"
                  :label="dict.value">{{ dict.label }}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="内容">
              <editor v-model="form.noticeContent" :min-height="192"/>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <notice-detail-view ref="noticeViewRef"/>
    <read-users-dialog ref="readUsersRef"/>
  </div>
</template>

<script>
import NoticeDetailView from '@/layout/components/HeaderNotice/DetailView'
import ReadUsersDialog from './ReadUsers'
import { listNotice, getNotice, delNotice, addNotice, updateNotice } from '@/api/system/notice'

/**
 * 组件名称：Notice
 * 功能说明：公告管理页面，支持公告增删改查及已读用户查看
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {Boolean} loading     - 遮罩层
 * @data {Array}   ids         - 选中数组
 * @data {Boolean} single      - 非单个禁用
 * @data {Boolean} multiple    - 非多个禁用
 * @data {Boolean} showSearch  - 显示搜索条件
 * @data {Number}  total       - 总条数
 * @data {Array}   noticeList  - 公告表格数据
 * @data {String}  title       - 弹出层标题
 * @data {Boolean} open        - 是否显示弹出层
 * @data {Object}  queryParams - 查询参数
 * @data {Object}  form        - 表单参数
 * @data {Object}  rules       - 表单校验
 *
 * 事件说明：
 * @event getList               - 查询公告列表
 * @event cancel                - 取消按钮操作
 * @event reset                 - 表单重置
 * @event handleQuery           - 搜索按钮操作
 * @event resetQuery            - 重置按钮操作
 * @event handleSelectionChange - 多选框选中数据
 * @event handleAdd             - 新增按钮操作
 * @event handleUpdate          - 修改按钮操作
 * @event submitForm            - 提交按钮操作
 * @event handleViewData        - 查看公告详情
 * @event handleReadUsers       - 查看已读用户
 * @event handleDelete          - 删除按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence   初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'Notice',
  components:
  {
    NoticeDetailView,
    ReadUsersDialog
  },
  dicts: ['sys_notice_status', 'sys_notice_type'],
  data()
  {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 公告表格数据
      noticeList: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        noticeTitle: undefined,
        createBy: undefined,
        status: undefined,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        noticeTitle: [{ required: true, message: '公告标题不能为空', trigger: 'blur' }],
        noticeType: [{ required: true, message: '公告类型不能为空', trigger: 'change' }],
      },
    }
  },
  created()
  {
    this.getList()
  },
  methods:
  {
    /**
     * 查询公告列表
     */
    getList()
    {
      this.loading = true
      listNotice(this.queryParams).then((response) => {
        this.noticeList = response.rows
        this.total = response.total
        this.loading = false
      })
    },
    /**
     * 取消按钮操作
     */
    cancel()
    {
      this.open = false
      this.reset()
    },
    /**
     * 表单重置
     */
    reset()
    {
      this.form = {
        noticeId: undefined,
        noticeTitle: undefined,
        noticeType: undefined,
        noticeContent: undefined,
        status: '0',
      }
      this.resetForm('form')
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
      this.resetForm('queryForm')
      this.handleQuery()
    },
    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的行数据
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.noticeId)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },
    /**
     * 新增按钮操作
     */
    handleAdd()
    {
      this.reset()
      this.open = true
      this.title = '添加公告'
    },
    /**
     * 修改按钮操作
     * @param {Object} row - 公告行数据
     */
    handleUpdate(row)
    {
      this.reset()
      const noticeId = row.noticeId || this.ids
      getNotice(noticeId).then((response) => {
        this.form = response.data
        this.open = true
        this.title = '修改公告'
      })
    },
    /**
     * 提交按钮操作
     */
    submitForm()
    {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          if (this.form.noticeId != undefined) {
            updateNotice(this.form).then(() => {
              this.$modal.msgSuccess('修改成功')
              this.open = false
              this.getList()
            })
          } else {
            addNotice(this.form).then(() => {
              this.$modal.msgSuccess('新增成功')
              this.open = false
              this.getList()
            })
          }
        }
      })
    },
    /**
     * 查看公告详情
     * @param {Object} row - 公告行数据
     */
    handleViewData(row)
    {
      this.$refs.noticeViewRef.open(row)
    },
    /**
     * 查看已读用户
     * @param {Object} row - 公告行数据
     */
    handleReadUsers(row)
    {
      this.$refs.readUsersRef.open(row)
    },
    /**
     * 删除按钮操作
     * @param {Object} row - 公告行数据
     */
    handleDelete(row)
    {
      const noticeIds = row.noticeId || this.ids
      this.$modal
        .confirm('是否确认删除公告编号为"' + noticeIds + '"的数据项？')
        .then(function () {
          return delNotice(noticeIds)
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('删除成功')
        })
        .catch(() => {})
    },
  },
}
</script>