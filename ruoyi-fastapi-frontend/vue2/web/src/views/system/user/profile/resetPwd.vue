<style lang="scss" scoped>
</style>

<template>
  <el-form
    ref="form"
    :model="user"
    :rules="formRules"
    label-width="80px">
    <el-form-item label="旧密码" prop="oldPassword">
      <el-input
        v-model="user.oldPassword"
        placeholder="请输入旧密码"
        type="password"
        show-password/>
    </el-form-item>
    <el-form-item
      label="新密码"
      prop="newPassword"
      :rules="infoPwdValidator">
      <el-input
        v-model="user.newPassword"
        placeholder="请输入新密码"
        type="password"
        show-password/>
    </el-form-item>
    <el-form-item label="确认密码" prop="confirmPassword">
      <el-input
        v-model="user.confirmPassword"
        placeholder="请确认新密码"
        type="password"
        show-password/>
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
 * 组件名称：ResetPwd
 * 功能说明：个人中心修改密码表单，校验并提交新旧密码
 * 研发时间：2024/01/22
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
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { updateUserPwd } from '@/api/system/user'
import passwordRule from '@/utils/passwordRule'

export default
{
  mixins: [passwordRule],

  data()
  {
    return {
      user: {
        oldPassword: undefined,
        newPassword: undefined,
        confirmPassword: undefined,
      },
    }
  },

  computed:
  {
    /**
     * 表单校验规则
     * @returns {Object} 校验规则对象
     */
    formRules()
    {
      return {
        oldPassword: [{ required: true, message: '旧密码不能为空', trigger: 'blur' }],
        confirmPassword: [
          { required: true, message: '确认密码不能为空', trigger: 'blur' },
          {
            validator: (rule, value, callback) =>
            {
              if (this.user.newPassword !== value)
              {
                callback(new Error('两次输入的密码不一致'))
              } else
              {
                callback()
              }
            },
            trigger: 'blur',
          },
        ],
      }
    },
  },

  methods:
  {
    /**
     * 提交修改密码
     */
    submit()
    {
      this.$refs['form'].validate((valid) =>
      {
        if (valid)
        {
          updateUserPwd(this.user.oldPassword, this.user.newPassword).then(() =>
          {
            this.$modal.msgSuccess('修改成功')
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