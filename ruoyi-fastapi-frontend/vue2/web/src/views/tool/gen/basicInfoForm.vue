<style lang="scss" scoped></style>

<template>
  <el-form
    ref="basicInfoForm"
    :model="info"
    :rules="rules"
    label-width="150px">
    <el-row>
      <el-col :span="12">
        <el-form-item label="表名称" prop="tableName">
          <el-input
            placeholder="请输入仓库名称"
            v-model="info.tableName"/>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="表描述" prop="tableComment">
          <el-input
            placeholder="请输入"
            v-model="info.tableComment"/>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="实体类名称" prop="className">
          <el-input
            placeholder="请输入"
            v-model="info.className"/>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="作者" prop="functionAuthor">
          <el-input
            placeholder="请输入"
            v-model="info.functionAuthor"/>
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="备注" prop="remark">
          <el-input
            type="textarea"
            :rows="3"
            v-model="info.remark"></el-input>
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>

<script>
/**
 * 组件名称：basicInfoForm
 * 功能说明：代码生成基本信息表单组件
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Object} info - 表基本信息
 *
 * 变量说明：
 * @data {Object} rules - 表单校验规则
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2024/01/22     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  props:
  {
    info: { type: Object, default: null },
  },

  data()
  {
    return {
      rules: {
        tableName: [{ required: true, message: '请输入表名称', trigger: 'blur' }],
        tableComment: [{ required: true, message: '请输入表描述', trigger: 'blur' }],
        className: [{ required: true, message: '请输入实体类名称', trigger: 'blur' }],
        functionAuthor: [{ required: true, message: '请输入作者', trigger: 'blur' }],
      },
    }
  }
}
</script>