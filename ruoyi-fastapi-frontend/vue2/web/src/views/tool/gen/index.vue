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
      <el-form-item label="表名称" prop="tableName">
        <el-input
          v-model="queryParams.tableName"
          placeholder="请输入表名称"
          clearable
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="表描述" prop="tableComment">
        <el-input
          v-model="queryParams.tableComment"
          placeholder="请输入表描述"
          clearable
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="数据源" prop="dataSourceName">
        <el-select
          v-model="queryParams.dataSourceName"
          placeholder="请选择数据源"
          clearable
          filterable
          style="width: 220px">
          <el-option
            v-for="source in dataSources"
            :key="source.name"
            :label="source.name + '（' + source.dbType + '）'"
            :value="source.name"/>
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          v-model="dateRange"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
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
          icon="el-icon-download"
          size="mini"
          :disabled="multiple"
          v-hasPermi="['tool:gen:code']"
          @click="handleGenTable">生成</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          v-hasRole="['admin']"
          @click="openCreateTable">创建</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          icon="el-icon-upload"
          size="mini"
          v-hasPermi="['tool:gen:import']"
          @click="openImportTable">导入</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          v-hasPermi="['tool:gen:edit']"
          @click="handleEditTable">修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          v-hasPermi="['tool:gen:remove']"
          @click="handleDelete">删除</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="tableList"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        align="center"
        width="55"></el-table-column>
      <el-table-column
        label="序号"
        type="index"
        width="50"
        align="center">
        <template slot-scope="scope">
          <span>{{ (queryParams.pageNum - 1) * queryParams.pageSize + scope.$index + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="表名称"
        align="center"
        prop="tableName"
        :show-overflow-tooltip="true"
        width="140"/>
      <el-table-column
        label="表描述"
        align="center"
        prop="tableComment"
        :show-overflow-tooltip="true"
        width="140"/>
      <el-table-column
        label="实体"
        align="center"
        prop="className"
        :show-overflow-tooltip="true"
        width="140"/>
      <el-table-column
        label="数据源"
        align="center"
        prop="dataSourceName"
        width="160"/>
      <el-table-column
        label="创建时间"
        align="center"
        prop="createTime"
        width="160"/>
      <el-table-column
        label="更新时间"
        align="center"
        prop="updateTime"
        width="160"/>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            icon="el-icon-view"
            v-hasPermi="['tool:gen:preview']"
            @click="handlePreview(scope.row)">预览</el-button>
          <el-button
            type="text"
            size="small"
            icon="el-icon-edit"
            v-hasPermi="['tool:gen:edit']"
            @click="handleEditTable(scope.row)">编辑</el-button>
          <el-button
            type="text"
            size="small"
            icon="el-icon-delete"
            v-hasPermi="['tool:gen:remove']"
            @click="handleDelete(scope.row)">删除</el-button>
          <el-button
            type="text"
            size="small"
            icon="el-icon-refresh"
            v-hasPermi="['tool:gen:edit']"
            @click="handleSynchDb(scope.row)">同步</el-button>
          <el-button
            type="text"
            size="small"
            icon="el-icon-download"
            v-hasPermi="['tool:gen:code']"
            @click="handleGenTable(scope.row)">生成代码</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"/>
    <!-- 预览界面 -->
    <el-dialog
      class="scrollbar"
      :title="preview.title"
      :visible.sync="preview.open"
      width="80%"
      top="5vh"
      append-to-body>
      <el-tabs v-model="preview.activeName">
        <el-tab-pane
          v-for="(value, key) in preview.data"
          :label="key.substring(key.lastIndexOf('/') + 1, key.indexOf('.jinja2'))"
          :name="key.substring(key.lastIndexOf('/') + 1, key.indexOf('.jinja2'))"
          :key="key">
          <el-link
            :underline="false"
            icon="el-icon-document-copy"
            v-clipboard:copy="value"
            v-clipboard:success="clipboardSuccess"
            style="float: right">复制</el-link>
          <pre><code class="hljs" v-html="highlightedCode(value, key)"></code></pre>
        </el-tab-pane>
      </el-tabs>
    </el-dialog>
    <import-table
      ref="import"
      :data-sources="dataSources"
      @ok="handleQuery"/>
    <create-table
      ref="create"
      :data-sources="dataSources"
      @ok="handleQuery"/>
  </div>
</template>

<script>
/**
 * 组件名称：Gen
 * 功能说明：代码生成页面，支持数据源表查询、导入、创建、生成代码与预览
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 变量说明：
 * @data {Boolean} loading             - 遮罩层
 * @data {String}  uniqueId            - 唯一标识符
 * @data {Array}   ids                 - 选中数组
 * @data {Array}   tableNames          - 选中表数组
 * @data {Array}   selectedSourceNames - 选中表的数据源数组
 * @data {Boolean} single              - 非单个禁用
 * @data {Boolean} multiple            - 非多个禁用
 * @data {Boolean} showSearch          - 显示搜索条件
 * @data {Number}  total               - 总条数
 * @data {Array}   tableList           - 表数据
 * @data {Array}   dataSources         - 数据源选项
 * @data {String}  dateRange           - 日期范围
 * @data {Object}  queryParams         - 查询参数
 * @data {Object}  preview             - 预览参数
 *
 * 事件说明：
 * @event getList               - 查询表集合
 * @event getDataSources        - 查询代码生成数据源选项
 * @event handleQuery           - 搜索按钮操作
 * @event handleGenTable        - 生成代码操作
 * @event handleSynchDb         - 同步数据库操作
 * @event openImportTable       - 打开导入表弹窗
 * @event openCreateTable       - 打开创建表弹窗
 * @event resetQuery            - 重置按钮操作
 * @event handlePreview         - 预览按钮
 * @event highlightedCode       - 高亮显示
 * @event clipboardSuccess      - 复制代码成功
 * @event handleSelectionChange - 多选框选中数据
 * @event handleEditTable       - 修改按钮操作
 * @event handleDelete          - 删除按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import {
  listDataSources,
  listTable,
  previewTable,
  delTable,
  genCode,
  synchDb,
} from '@/api/tool/gen'
import importTable from './importTable'
import createTable from './createTable'
import hljs from 'highlight.js/lib/highlight'
import 'highlight.js/styles/github-gist.css'
hljs.registerLanguage('py', require('highlight.js/lib/languages/python'))
hljs.registerLanguage('html', require('highlight.js/lib/languages/xml'))
hljs.registerLanguage('vue', require('highlight.js/lib/languages/xml'))
hljs.registerLanguage('javascript', require('highlight.js/lib/languages/javascript'))
hljs.registerLanguage('sql', require('highlight.js/lib/languages/sql'))

export default
{
  name: 'Gen',

  components:
  {
    importTable,
    createTable
  },

  data()
  {
    return {
      // 遮罩层
      loading: true,
      // 唯一标识符
      uniqueId: '',
      // 选中数组
      ids: [],
      // 选中表数组
      tableNames: [],
      // 选中表的数据源数组
      selectedSourceNames: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 表数据
      tableList: [],
      // 数据源选项
      dataSources: [],
      // 日期范围
      dateRange: '',
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        tableName: undefined,
        tableComment: undefined,
        dataSourceName: undefined,
      },
      // 预览参数
      preview: {
        open: false,
        title: '代码预览',
        data: {},
        activeName: 'do.py',
      },
    }
  },

  created()
  {
    this.getList()
    this.getDataSources()
  },

  activated()
  {
    const time = this.$route.query.t
    if (time != null && time != this.uniqueId)
    {
      this.uniqueId = time
      this.queryParams.pageNum = Number(this.$route.query.pageNum)
      this.getList()
    }
  },

  methods:
  {
    /**
     * 查询表集合
     */
    getList()
    {
      this.loading = true
      listTable(this.addDateRange(this.queryParams, this.dateRange)).then((response) =>
      {
        this.tableList = response.rows
        this.total = response.total
        this.loading = false
      })
    },

    /**
     * 查询代码生成数据源选项
     */
    getDataSources()
    {
      listDataSources().then((response) =>
      {
        this.dataSources = response.data || []
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
     * 生成代码操作
     * @param {Object} row - 表行数据
     */
    handleGenTable(row)
    {
      const tableNames = row.tableName || this.tableNames
      if (tableNames == '')
      {
        this.$modal.msgError('请选择要生成的数据')
        return
      }
      const sourceNames = row.tableName ? [row.dataSourceName] : this.selectedSourceNames
      const uniqueSourceNames = [...new Set(sourceNames.filter(Boolean))]
      if (uniqueSourceNames.length !== 1)
      {
        this.$modal.msgError('请选择同一数据源下的生成配置')
        return
      }
      const dataSourceName = uniqueSourceNames[0]
      if (row.genType === '1')
      {
        genCode(row.tableName, dataSourceName).then(() =>
        {
          this.$modal.msgSuccess('成功生成到自定义路径：' + row.genPath)
        })
      }
      else
      {
        const zipName = Array.isArray(tableNames) ? 'vfadmin.zip' : tableNames + '.zip'
        const tables = encodeURIComponent(
          Array.isArray(tableNames) ? tableNames.join(',') : tableNames
        )
        this.$download.zip(
          '/tool/gen/batchGenCode?tables=' +
            tables +
            '&dataSourceName=' +
            encodeURIComponent(dataSourceName),
          zipName
        )
      }
    },

    /**
     * 同步数据库操作
     * @param {Object} row - 表行数据
     */
    handleSynchDb(row)
    {
      const tableName = row.tableName
      const dataSourceName = row.dataSourceName
      this.$modal
        .confirm('确认要强制同步"' + tableName + '"表结构吗？')
        .then(function ()
        {
          return synchDb(tableName, dataSourceName)
        })
        .then(() =>
        {
          this.$modal.msgSuccess('同步成功')
        })
        .catch(() => {})
    },

    /**
     * 打开导入表弹窗
     */
    openImportTable()
    {
      this.$refs.import.show()
    },

    /**
     * 打开创建表弹窗
     */
    openCreateTable()
    {
      this.$refs.create.show()
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
     * 预览按钮
     * @param {Object} row - 表行数据
     */
    handlePreview(row)
    {
      previewTable(row.tableId).then((response) =>
      {
        this.preview.data = response.data
        this.preview.open = true
        this.preview.activeName = 'do.py'
      })
    },

    /**
     * 高亮显示
     * @param {String} code - 代码内容
     * @param {String} key - 模板标识
     * @returns {String} 高亮后的 HTML 内容
     */
    highlightedCode(code, key)
    {
      const vmName = key.substring(key.lastIndexOf('/') + 1, key.indexOf('.jinja2'))
      var language = vmName.substring(vmName.indexOf('.') + 1, vmName.length)
      const result = hljs.highlight(language, code || '', true)
      return result.value || '&nbsp;'
    },

    /**
     * 复制代码成功
     */
    clipboardSuccess()
    {
      this.$modal.msgSuccess('复制成功')
    },

    /**
     * 多选框选中数据
     * @param {Array} selection - 选中的行数据
     */
    handleSelectionChange(selection)
    {
      this.ids = selection.map((item) => item.tableId)
      this.tableNames = selection.map((item) => item.tableName)
      this.selectedSourceNames = selection.map((item) => item.dataSourceName)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },

    /**
     * 修改按钮操作
     * @param {Object} row - 表行数据
     */
    handleEditTable(row)
    {
      const tableId = row.tableId || this.ids[0]
      const tableName = row.tableName || this.tableNames[0]
      const params = { pageNum: this.queryParams.pageNum }
      this.$tab.openPage(
        '修改[' + tableName + ']生成配置',
        '/tool/gen-edit/index/' + tableId,
        params
      )
    },

    /**
     * 删除按钮操作
     * @param {Object} row - 表行数据
     */
    handleDelete(row)
    {
      const tableIds = row.tableId || this.ids
      this.$modal
        .confirm('是否确认删除表编号为"' + tableIds + '"的数据项？')
        .then(function ()
        {
          return delTable(tableIds)
        })
        .then(() =>
        {
          this.getList()
          this.$modal.msgSuccess('删除成功')
        })
        .catch(() => {})
    },
  },
}
</script>