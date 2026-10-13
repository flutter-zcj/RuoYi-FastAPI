<style lang="scss" scoped></style>

<template>
  <div class="app-container">
    <el-form
      ref="queryForm"
      :model="queryParams"
      size="small"
      :inline="true"
      v-show="showSearch"
      label-width="68px">
      <el-form-item label="字典名称" prop="dictName">
        <el-input
          v-model="queryParams.dictName"
          placeholder="请输入字典名称"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="字典类型" prop="dictType">
        <el-input
          v-model="queryParams.dictType"
          placeholder="请输入字典类型"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="字典状态"
          clearable
          style="width: 240px">
          <el-option
            v-for="dict in dict.type.sys_normal_disable"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          v-model="dateRange"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
          :aria-label="`日期范围（${getDisplayTimezone()}）`"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"></el-date-picker>
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
          v-hasPermi="['system:dict:add']"
          @click="handleAdd">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          v-hasPermi="['system:dict:edit']"
          @click="handleUpdate">修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          v-hasPermi="['system:dict:remove']"
          @click="handleDelete">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          v-hasPermi="['system:dict:export']"
          @click="handleExport">导出</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-refresh"
          size="mini"
          v-hasPermi="['system:dict:remove']"
          @click="handleRefreshCache">刷新缓存</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="typeList"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55"
        align="center"/>
      <el-table-column
        label="字典编号"
        align="center"
        prop="dictId"/>
      <el-table-column
        label="字典名称"
        align="center"
        prop="dictName"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="字典类型"
        align="center"
        :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <a
            class="link-type"
            style="cursor: pointer"
            @click="handleViewData(scope.row)">{{ scope.row.dictType }}</a>
        </template>
      </el-table-column>
      <el-table-column
        label="状态"
        align="center"
        prop="status">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_normal_disable" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column
        label="备注"
        align="center"
        prop="remark"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="创建时间"
        align="center"
        prop="createTime"
        width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
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
            icon="el-icon-edit"
            v-hasPermi="['system:dict:edit']"
            @click="handleUpdate(scope.row)">修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-s-operation"
            v-hasPermi="['system:dict:edit']"
            @click="handleDataList(scope.row)">列表</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            v-hasPermi="['system:dict:remove']"
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

    <!-- 添加或修改参数配置对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="open"
      width="500px"
      append-to-body>
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        label-width="80px">
        <el-form-item label="字典名称" prop="dictName">
          <el-input v-model="form.dictName" placeholder="请输入字典名称"/>
        </el-form-item>
        <el-form-item prop="dictType">
          <el-input
            v-model="form.dictType"
            placeholder="请输入字典类型"
            maxlength="100"/>
          <span slot="label">
            <el-tooltip content="数据存储中的Key值，如：sys_user_sex" placement="top">
              <i class="el-icon-question"></i>
            </el-tooltip>
            字典类型
          </span>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in dict.type.sys_normal_disable"
              :key="dict.value"
              :label="dict.value">{{ dict.label }}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            placeholder="请输入内容"></el-input>
        </el-form-item>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <dict-data-drawer :visible.sync="drawerVisible" :row="drawerRow"/>
  </div>
</template>

<script>
/**
 * 组件名称：Dict
 * 功能说明：字典类型管理页面，支持字典类型的增删改查、导出与缓存刷新
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} loading       - 遮罩层状态
 * @data {Array}   ids           - 选中数组
 * @data {Boolean} single        - 非单个禁用状态
 * @data {Boolean} multiple      - 非多个禁用状态
 * @data {Boolean} showSearch    - 是否显示搜索条件
 * @data {Number}  total         - 总条数
 * @data {Array}   typeList      - 字典表格数据
 * @data {String}  title         - 弹出层标题
 * @data {Boolean} open          - 是否显示弹出层
 * @data {Boolean} drawerVisible - 字典数据抽屉状态
 * @data {Object}  drawerRow     - 字典数据信息
 * @data {Array}   dateRange     - 日期范围
 * @data {Object}  queryParams   - 查询参数
 * @data {Object}  form          - 表单参数
 * @data {Object}  rules         - 表单校验规则
 *
 * 事件说明：
 * @event getDisplayTimezone     - 获取显示时区
 * @event getList                - 查询字典类型列表
 * @event cancel                 - 取消按钮操作
 * @event reset                  - 表单重置
 * @event handleQuery            - 搜索按钮操作
 * @event resetQuery             - 重置按钮操作
 * @event handleAdd              - 新增按钮操作
 * @event handleSelectionChange  - 多选框选中数据
 * @event handleViewData         - 字典数据抽屉显示信息
 * @event handleDataList         - 字典数据列表页面
 * @event handleUpdate           - 修改按钮操作
 * @event submitForm             - 提交按钮操作
 * @event handleDelete           - 删除按钮操作
 * @event handleExport           - 导出按钮操作
 * @event handleRefreshCache     - 刷新缓存按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { getDisplayTimezone } from '@/utils/time'
import DictDataDrawer from './detail'
import {
  listType,
  getType,
  delType,
  addType,
  updateType,
  refreshCache,
} from '@/api/system/dict/type'

export default
{
  name: 'Dict',

  components:
  {
    DictDataDrawer
  },

  dicts: ['sys_normal_disable'],

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
      // 字典表格数据
      typeList: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // 字典数据抽屉状态
      drawerVisible: false,
      // 字典数据信息
      drawerRow: {},
      // 日期范围
      dateRange: [],
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        dictName: undefined,
        dictType: undefined,
        status: undefined,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        dictName: [{ required: true, message: '字典名称不能为空', trigger: 'blur' }],
        dictType: [{ required: true, message: '字典类型不能为空', trigger: 'blur' }],
      },
    }
  },

  /**
   * 时区切换后重新解释日期筛选，保留当前表单
   */
  created()
  {
    // 时区切换后重新解释日期筛选，保留当前表单。
    this.$watch(getDisplayTimezone, () => {
      if (this.dateRange && this.dateRange.length) this.handleQuery()
    })
    this.getList()
  },

  methods:
  {
    /**
     * 获取显示时区
     */
    getDisplayTimezone,

    /**
     * 查询字典类型列表
     */
    getList()
    {
      this.loading = true
      listType(this.addDateRange(this.queryParams, this.dateRange)).then((response) => {
        this.typeList = response.rows
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
        dictId: undefined,
        dictName: undefined,
        dictType: undefined,
        status: '0',
        remark: undefined,
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
      this.dateRange = []
      this.resetForm('queryForm')
      this.handleQuery()
    },

    /**
     * 新增按钮操作
     */
    handleAdd()
    {
      this.reset()
      this.open = true
      this.title = '添加字典类型'
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中行数据
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.dictId)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },

    /**
     * 字典数据抽屉显示信息
     * @param {Object} row - 当前字典类型行数据
     */
    handleViewData(row)
    {
      this.drawerRow = row
      this.drawerVisible = true
    },

    /**
     * 字典数据列表页面
     * @param {Object} row - 当前字典类型行数据
     */
    handleDataList(row)
    {
      this.$tab.openPage('字典数据', '/system/dict-data/index/' + row.dictId)
    },

    /**
     * 修改按钮操作
     * @param {Object} row - 当前字典类型行数据
     */
    handleUpdate(row)
    {
      this.reset()
      const dictId = row.dictId || this.ids
      getType(dictId).then((response) => {
        this.form = response.data
        this.open = true
        this.title = '修改字典类型'
      })
    },

    /**
     * 提交按钮操作
     */
    submitForm()
    {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          if (this.form.dictId != undefined) {
            updateType(this.form).then(() => {
              this.$modal.msgSuccess('修改成功')
              this.open = false
              this.getList()
            })
          } else {
            addType(this.form).then(() => {
              this.$modal.msgSuccess('新增成功')
              this.open = false
              this.getList()
            })
          }
        }
      })
    },

    /**
     * 删除按钮操作
     * @param {Object} row - 当前字典类型行数据
     */
    handleDelete(row)
    {
      const dictIds = row.dictId || this.ids
      this.$modal
        .confirm('是否确认删除字典编号为"' + dictIds + '"的数据项？')
        .then(function () {
          return delType(dictIds)
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('删除成功')
        })
        .catch(() => {})
    },

    /**
     * 导出按钮操作
     */
    handleExport()
    {
      this.download(
        'system/dict/type/export',
        {
          ...this.queryParams,
        },
        `type_${new Date().getTime()}.xlsx`
      )
    },

    /**
     * 刷新缓存按钮操作
     */
    handleRefreshCache()
    {
      refreshCache().then(() => {
        this.$modal.msgSuccess('刷新成功')
        this.$store.dispatch('dict/cleanDict')
      })
    },
  },
}
</script>