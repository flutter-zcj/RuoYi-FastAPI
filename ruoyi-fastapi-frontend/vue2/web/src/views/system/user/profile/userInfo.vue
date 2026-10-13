<style lang="scss" scoped>
</style>

<template>
  <el-form
    ref="form"
    :model="form"
    :rules="rules"
    label-width="80px">
    <el-form-item label="用户昵称" prop="nickName">
      <el-input v-model="form.nickName" maxlength="30" />
    </el-form-item>
    <el-form-item label="手机号码" prop="phonenumber">
      <el-input v-model="form.phonenumber" maxlength="11" />
    </el-form-item>
    <el-form-item label="邮箱" prop="email">
      <el-input v-model="form.email" maxlength="50" />
    </el-form-item>
    <el-form-item label="性别">
      <el-radio-group v-model="form.sex">
        <el-radio label="0">男</el-radio>
        <el-radio label="1">女</el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item>
      <el-button
        type="primary"
        size="mini"
        @click="submit">保存</el-button>
      <el-button
        type="danger"
        size="mini"
        @click="close">关闭</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
/**
 * 组件名称：UserInfo
 * 功能说明：个人中心-基本资料表单，展示并修改用户昵称、手机号码、邮箱与性别
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Object} user - 用户信息对象
 *
 * 变量说明：
 *
 * 事件说明：
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { updateUserProfile } from '@/api/system/user'

export default
{
  props:
  {
    user: { type: Object }
  },

  data()
  {
    return {
      form: {},
      // 表单校验
      rules: {
        nickName: [{ required: true, message: '用户昵称不能为空', trigger: 'blur' }],
        email: [
          { required: true, message: '邮箱地址不能为空', trigger: 'blur' },
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change'],
          },
        ],
        phonenumber: [
          { required: true, message: '手机号码不能为空', trigger: 'blur' },
          {
            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
            message: '请输入正确的手机号码',
            trigger: 'blur',
          },
        ],
      },
    }
  },

  watch:
  {
    user:
    {
      /**
       * 用户信息变化时同步表单数据
       * @param {Object} user - 用户信息对象
       */
      handler(user)
      {
        if (user)
        {
          this.form = {
            nickName: user.nickName,
            phonenumber: user.phonenumber,
            email: user.email,
            sex: user.sex,
          }
        }
      },
      immediate: true,
    },
  },

  methods:
  {
    /**
     * 提交保存用户资料
     */
    submit()
    {
      this.$refs['form'].validate((valid) =>
      {
        if (valid)
        {
          updateUserProfile(this.form).then(() =>
          {
            this.$modal.msgSuccess('修改成功')
            this.user.phonenumber = this.form.phonenumber
            this.user.email = this.form.email
          })
        }
      })
    },

    /**
     * 关闭当前页签
     */
    close()
    {
      this.$tab.closePage()
    },
  },
}
</script>