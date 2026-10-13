<style lang="scss" scoped></style>

<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      label-width="68px">
      <el-form-item label="登录地址" prop="ipaddr">
        <el-input
          v-model="queryParams.ipaddr"
          placeholder="请输入登录地址"
          clearable
          @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="用户名称" prop="userName">
        <el-input
          v-model="queryParams.userName"
          placeholder="请输入用户名称"
          clearable
          @keyup.enter.native="handleQuery"/>
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
    <el-table
      v-loading="loading"
      :data="list.slice((pageNum - 1) * pageSize, pageNum * pageSize)"
      style="width: 100%">
      <el-table-column
        label="序号"
        type="index"
        align="center">
        <template slot-scope="scope">
          <span>{{ (pageNum - 1) * pageSize + scope.$index + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="会话编号"
        align="center"
        prop="tokenId"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="登录名称"
        align="center"
        prop="userName"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="部门名称"
        align="center"
        prop="deptName"/>
      <el-table-column
        label="主机"
        align="center"
        prop="ipaddr"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="登录地点"
        align="center"
        prop="loginLocation"
        :show-overflow-tooltip="true"/>
      <el-table-column
        label="浏览器"
        align="center"
        prop="browser"/>
      <el-table-column
        label="操作系统"
        align="center"
        prop="os"/>
      <el-table-column
        label="登录时间"
        align="center"
        prop="loginTime"
        width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.loginTime) }}</span>
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
            icon="el-icon-delete"
            v-hasPermi="['monitor:online:forceLogout']"
            @click="handleForceLogout(scope.row)">强退</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="pageNum"
      :limit.sync="pageSize"/>
  </div>
</template>

<script>
/**
 * 组件名称：Online
 * 功能说明：在线用户监控页面，支持在线会话查询与强制退出
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Boolean} loading     - 遮罩层
 * @data {Number}  total       - 总条数
 * @data {Array}   list        - 表格数据
 * @data {Number}  pageNum     - 当前页码
 * @data {Number}  pageSize    - 每页条数
 * @data {Object}  queryParams - 查询参数
 *
 * 事件说明：
 * @event getList           - 查询登录日志列表
 * @event handleQuery       - 搜索按钮操作
 * @event resetQuery        - 重置按钮操作
 * @event handleForceLogout - 强退按钮操作
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { list, forceLogout } from '@/api/monitor/online'

export default
{
  name: 'Online',

  data()
  {
    return {
      // 遮罩层
      loading: true,
      // 总条数
      total: 0,
      // 表格数据
      list: [],
      pageNum: 1,
      pageSize: 10,
      // 查询参数
      queryParams: {
        ipaddr: undefined,
        userName: undefined,
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
     * 查询登录日志列表
     */
    getList()
    {
      this.loading = true
      list(this.queryParams).then((response) => {
        this.list = response.rows
        this.total = response.total
        this.loading = false
      })
    },

    /**
     * 搜索按钮操作
     */
    handleQuery()
    {
      this.pageNum = 1
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
     * 强退按钮操作
     */
    handleForceLogout(row)
    {
      this.$modal
        .confirm('是否确认强退名称为"' + row.userName + '"的用户？')
        .then(function () {
          return forceLogout(row.tokenId)
        })
        .then(() => {
          this.getList()
          this.$modal.msgSuccess('强退成功')
        })
        .catch(() => {})
    },
  },
}
</script>