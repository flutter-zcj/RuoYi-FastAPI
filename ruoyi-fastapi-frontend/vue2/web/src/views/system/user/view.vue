<style lang="scss" scoped>
</style>

<template>
  <el-drawer
    title="用户信息详情"
    :visible.sync="visible"
    direction="rtl"
    size="68%"
    append-to-body
    :before-close="handleClose"
    custom-class="detail-drawer">
    <div class="drawer-content" v-loading="loading">
      <!-- 基本信息 -->
      <h4 class="section-header">基本信息</h4>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">用户名称：</label>
            <span class="info-value plaintext">{{ info.nickName }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">归属部门：</label>
            <span class="info-value plaintext">{{ info.dept && info.dept.deptName }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">手机号码：</label>
            <span class="info-value plaintext">{{ info.phonenumber }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">邮箱：</label>
            <span class="info-value plaintext">{{ info.email }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">登录账号：</label>
            <span class="info-value plaintext">{{ info.userName }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">用户状态：</label>
            <span class="info-value plaintext">
              <el-tag size="small" :type="info.status === '0' ? 'success' : 'danger'">{{ info.status === '0' ? '正常' : '停用' }}</el-tag>
            </span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">岗位：</label>
            <span class="info-value plaintext">{{ postNames || '无岗位' }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">用户性别：</label>
            <span class="info-value plaintext">{{ sexLabel }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="24">
          <div class="info-item full-width">
            <label class="info-label">角色：</label>
            <span class="info-value plaintext">{{ roleNames || '无角色' }}</span>
          </div>
        </el-col>
      </el-row>
      <!-- 其他信息 -->
      <h4 class="section-header">其他信息</h4>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">创建者：</label>
            <span class="info-value plaintext">{{ info.createBy }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">创建时间：</label>
            <span class="info-value plaintext">{{ parseTime(info.createTime) }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">更新者：</label>
            <span class="info-value plaintext">{{ info.updateBy }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">更新时间：</label>
            <span class="info-value plaintext">{{ parseTime(info.updateTime) }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">最后登录IP：</label>
            <span class="info-value plaintext">{{ info.loginIp }}</span>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="info-item">
            <label class="info-label">最后登录时间：</label>
            <span class="info-value plaintext">{{ parseTime(info.loginDate) }}</span>
          </div>
        </el-col>
      </el-row>
      <el-row class="mb8" :gutter="20">
        <el-col :span="24">
          <div class="info-item full-width">
            <label class="info-label">备注：</label>
            <span class="info-value plaintext">{{ info.remark }}</span>
          </div>
        </el-col>
      </el-row>
    </div>
  </el-drawer>
</template>

<script>
/**
 * 组件名称：UserView
 * 功能说明：用户信息详情抽屉，展示用户基本资料、部门、岗位、角色与登录信息
 * 研发时间：2026/07/31
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

import { getUser } from '@/api/system/user'

export default
{
  name: 'UserViewDrawer',
  dicts: ['sys_user_sex'],

  data()
  {
    return {
      visible: false,
      loading: false,
      info: {},
      postOptions: [],
      roleOptions: [],
    }
  },

  computed:
  {
    /**
     * 用户性别显示文本
     * @returns {String} 性别标签
     */
    sexLabel()
    {
      return this.selectDictLabel(this.dict.type.sys_user_sex, this.info.sex) || '-'
    },

    /**
     * 岗位名称集合
     * @returns {String} 岗位名称
     */
    postNames()
    {
      if (!this.postOptions.length) return ''
      const ids = this.info.postIds || []
      return (
        this.postOptions
          .filter((p) => ids.includes(p.postId))
          .map((p) => p.postName)
          .join('、') || ''
      )
    },

    /**
     * 角色名称集合
     * @returns {String} 角色名称
     */
    roleNames()
    {
      if (!this.roleOptions.length) return ''
      const ids = this.info.roleIds || []
      return (
        this.roleOptions
          .filter((r) => ids.includes(r.roleId))
          .map((r) => r.roleName)
          .join('、') || ''
      )
    },
  },

  methods:
  {
    /**
     * 打开详情抽屉并加载用户信息
     * @param {Number} userId - 用户ID
     */
    open(userId)
    {
      this.visible = true
      this.loading = true
      getUser(userId)
        .then((res) =>
        {
          this.info = res.data || {}
          this.postOptions = res.posts || []
          this.roleOptions = res.roles || []
          this.info.postIds = res.postIds || []
          this.info.roleIds = res.roleIds || []
        })
        .finally(() =>
        {
          this.loading = false
        })
    },

    /**
     * 关闭抽屉
     */
    handleClose()
    {
      this.visible = false
    },
  },
}
</script>