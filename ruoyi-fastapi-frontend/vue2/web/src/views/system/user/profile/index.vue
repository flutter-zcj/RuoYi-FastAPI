<style lang="scss" scoped>
</style>

<template>
  <div class="app-container">
    <el-row :gutter="20">
      <el-col :span="6" :xs="24">
        <el-card class="box-card">
          <div class="clearfix" slot="header">
            <span>个人信息</span>
          </div>
          <div>
            <div class="text-center">
              <userAvatar />
            </div>
            <ul class="list-group list-group-striped">
              <li class="list-group-item">
                <svg-icon icon-class="user" />用户名称
                <div class="pull-right">{{ user.userName }}</div>
              </li>
              <li class="list-group-item">
                <svg-icon icon-class="phone" />手机号码
                <div class="pull-right">{{ user.phonenumber }}</div>
              </li>
              <li class="list-group-item">
                <svg-icon icon-class="email" />用户邮箱
                <div class="pull-right">{{ user.email }}</div>
              </li>
              <li class="list-group-item">
                <svg-icon icon-class="tree" />所属部门
                <div class="pull-right" v-if="user.dept">
                  {{ user.dept.deptName }} / {{ postGroup }}
                </div>
              </li>
              <li class="list-group-item">
                <svg-icon icon-class="peoples" />所属角色
                <div class="pull-right">{{ roleGroup }}</div>
              </li>
              <li class="list-group-item">
                <svg-icon icon-class="date" />创建日期
                <div class="pull-right">{{ parseTime(user.createTime) || '-' }}</div>
              </li>
            </ul>
          </div>
        </el-card>
      </el-col>
      <el-col :span="18" :xs="24">
        <el-card>
          <div class="clearfix" slot="header">
            <span>基本资料</span>
          </div>
          <el-tabs v-model="selectedTab">
            <el-tab-pane label="基本资料" name="userinfo">
              <userInfo :user="user" />
            </el-tab-pane>
            <el-tab-pane label="修改密码" name="resetPwd">
              <resetPwd />
            </el-tab-pane>
            <el-tab-pane label="时区设置" name="timezone">
              <timezoneSettings :user="user" @saved="$set(user, 'timeZone', $event)" />
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
/**
 * 组件名称：Profile
 * 功能说明：个人中心页面，展示个人资料并切换基本资料、修改密码、时区设置页签
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

import timezoneSettings from './timezoneSettings.vue'
import userAvatar from './userAvatar'
import userInfo from './userInfo'
import resetPwd from './resetPwd'
import { getUserProfile } from '@/api/system/user'

export default
{
  name: 'Profile',
  components: { userAvatar, userInfo, resetPwd, timezoneSettings },

  data()
  {
    return {
      user: {},
      roleGroup: {},
      postGroup: {},
      selectedTab: 'userinfo',
    }
  },

  created()
  {
    const activeTab = this.$route.params && this.$route.params.activeTab
    if (activeTab)
    {
      this.selectedTab = activeTab
    }
    this.getUser()
  },

  methods:
  {
    /**
     * 获取当前用户个人资料
     */
    getUser()
    {
      getUserProfile().then((response) =>
      {
        this.user = response.data
        this.$store.dispatch('ApplyTimezone', response.data.timeZone)
        this.roleGroup = response.roleGroup
        this.postGroup = response.postGroup
      })
    },
  },
}
</script>