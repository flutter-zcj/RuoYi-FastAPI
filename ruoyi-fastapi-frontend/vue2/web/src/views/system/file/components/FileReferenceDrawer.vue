<style lang="scss" scoped>
.drawer-content { padding: 0 20px 20px; }
</style>

<template>
  <el-drawer
    :title="`业务引用 - ${fileName}`"
    :visible.sync="visible"
    size="55%"
    append-to-body>
    <div class="drawer-content">
      <el-alert
        class="mb8"
        title="存在业务引用时文件不能移入回收站，请先在对应业务中解除文件引用。"
        type="warning"
        :closable="false"
        show-icon />
      <el-table v-loading="loading" :data="referenceList">
        <el-table-column
          label="业务类型"
          align="center"
          prop="businessType"
          width="130"
          :show-overflow-tooltip="true" />
        <el-table-column
          label="业务名称"
          align="left"
          prop="businessName"
          min-width="180"
          :show-overflow-tooltip="true">
          <template slot-scope="scope">{{ scope.row.businessName || '-' }}</template>
        </el-table-column>
        <el-table-column
          label="业务ID"
          align="center"
          prop="businessId"
          min-width="160"
          :show-overflow-tooltip="true" />
        <el-table-column
          label="保留到期"
          align="center"
          prop="retentionExpireTime"
          width="180">
          <template slot-scope="scope">
            {{
              scope.row.retentionExpireTime ? parseTime(scope.row.retentionExpireTime) : '永久保留'
            }}
          </template>
        </el-table-column>
        <el-table-column
          label="来源"
          align="center"
          width="100">
          <template slot-scope="scope">
            <el-tag
              v-if="scope.row.legacy"
              type="info"
              effect="plain">
              兼容字段
            </el-tag>
            <el-tag
              v-else
              type="success"
              effect="plain">引用关系</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="登记人"
          align="center"
          prop="createBy"
          width="110"
          :show-overflow-tooltip="true">
          <template slot-scope="scope">{{ scope.row.createBy || '-' }}</template>
        </el-table-column>
        <el-table-column
          label="登记时间"
          align="center"
          prop="createTime"
          width="180">
          <template slot-scope="scope">
            {{ parseTime(scope.row.createTime) || '-' }}
          </template>
        </el-table-column>
      </el-table>
    </div>
  </el-drawer>
</template>

<script>
import { listFileReference } from '@/api/system/file'

/**
 * 组件名称：FileReferenceDrawer
 * 功能说明：文件业务引用抽屉组件，展示引用该文件的业务信息
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
  name: 'FileReferenceDrawer',

  data()
  {
    return {
      visible: false,
      loading: false,
      fileName: '',
      referenceList: [],
    }
  },

  methods:
  {
    /**
     * 打开抽屉并查询文件业务引用
     * @param {Object} row - 当前行数据
     */
    open(row)
    {
      this.fileName = row.originalName
      this.visible = true
      this.loading = true
      listFileReference(row.fileId)
        .then((response) =>
        {
          this.referenceList = response.data
        })
        .finally(() =>
        {
          this.loading = false
        })
    }
  }
}
</script>