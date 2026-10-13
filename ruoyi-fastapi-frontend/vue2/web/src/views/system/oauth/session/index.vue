<style lang="scss" scoped>
.result-count { font-size: 13px; color: var(--oauth-text-color-regular); }
.result-count strong { font-size: 16px; color: var(--oauth-text-color-primary); }
.person-cell, .detail-hero { display: flex; align-items: center; gap: 10px; }
.person-mark { display: grid; place-items: center; flex: 0 0 32px; width: 32px; height: 32px; background: var(--oauth-color-primary-light-9); color: var(--oauth-color-primary); border-radius: 50%; }
.person-mark.large { flex-basis: 42px; width: 42px; height: 42px; }
.person-cell strong, .person-cell small, .stacked-cell strong, .stacked-cell small { display: block; }
.person-cell strong, .stacked-cell strong { color: var(--oauth-text-color-primary); }
.person-cell small, .stacked-cell small { margin-top: 3px; font-size: 12px; color: var(--oauth-text-color-secondary); }
.tag-summary { display: flex; flex-wrap: wrap; align-items: center; gap: 5px; }
.muted-value { font-size: 12px; color: var(--oauth-text-color-secondary); }
.impact-summary { display: flex; gap: 12px; margin-bottom: 20px; padding: 14px 16px; border: 1px solid var(--oauth-color-warning-light-7); background: var(--oauth-color-warning-light-9); color: var(--oauth-color-warning-dark-2); border-radius: 8px; }
.impact-summary > .el-icon { flex: 0 0 auto; margin-top: 2px; font-size: 18px; }
.impact-summary strong { color: var(--oauth-text-color-primary); }
.impact-summary p { margin: 5px 0 0; font-size: 13px; line-height: 1.6; color: var(--oauth-text-color-regular); }
.reason-label { display: block; margin-bottom: 8px; font-weight: 600; color: var(--oauth-text-color-primary); }
.detail-hero { padding: 0 0 20px; border-bottom: 1px solid var(--oauth-border-color-lighter); }
.detail-hero > div { flex: 1; }
.detail-hero strong { font-size: 17px; color: var(--oauth-text-color-primary); }
.detail-hero p { margin: 5px 0 0; color: var(--oauth-text-color-secondary); }
.detail-section { padding: 18px 0; border-bottom: 1px solid var(--oauth-border-color-lighter); }
.detail-section h4 { margin: 0 0 10px; color: var(--oauth-text-color-primary); }
.detail-descriptions { margin-top: 18px; }
.technical-details { margin-top: 16px; }
.technical-details dl { display: grid; grid-template-columns: 100px minmax(0, 1fr); margin: 0; }
.technical-details dt, .technical-details dd { padding: 6px 0; }
.technical-details dt { color: var(--oauth-text-color-secondary); }
.technical-details dd { margin: 0; font-family: ui-monospace, SFMono-Regular, Menlo, monospace; color: var(--oauth-text-color-primary); overflow-wrap: anywhere; }
</style>

<template>
  <div class="oauth-route-page">
    <PageFrame
      section="访问控制"
      title="登录会话"
      description="查看正在使用统一登录的用户、来源位置和关联应用，并强制结束异常会话。"
      filter-hint="默认只显示仍然在线的会话">
      <template #actions>
        <el-button
          type="danger"
          plain
          icon="el-icon-switch-button"
          :disabled="!selected.length"
          v-hasPermi="['system:oauthSession:revoke']"
          @click="openRevoke()">强制下线{{ selected.length ? `（${selected.length}）` : '' }}</el-button>
      </template>
      <template #filters>
        <el-form
          ref="queryRef"
          :model="queryParams"
          :inline="true"
          v-show="showSearch">
          <el-form-item label="用户" prop="userId">
            <el-input
              v-model="queryParams.userId"
              clearable
              placeholder="用户编号"
              style="width: 200px"/>
          </el-form-item>
          <el-form-item label="来源地址" prop="ipAddress">
            <el-input
              v-model="queryParams.ipAddress"
              clearable
              placeholder="IP 地址"
              style="width: 200px"/>
          </el-form-item>
          <el-form-item label="状态" prop="status">
            <el-select
              v-model="queryParams.status"
              clearable
              placeholder="全部状态"
              style="width: 200px">
              <el-option label="在线" value="active"/>
              <el-option label="已下线" value="revoked"/>
              <el-option label="已过期" value="expired"/>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              icon="el-icon-search"
              @click="handleQuery">查找</el-button>
            <el-button icon="el-icon-refresh" @click="resetQuery">清空</el-button>
          </el-form-item>
        </el-form>
      </template>
      <template #toolbar>
        <span class="result-count">找到 <strong>{{ total }}</strong> 个会话</span>
        <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"/>
      </template>

      <el-table
        v-loading="loading"
        :data="rows"
        row-key="sid"
        @selection-change="selected = $event">
        <el-table-column
          type="selection"
          width="48"
          :selectable="(row) => row.status !== 'revoked'"/>
        <el-table-column label="用户" min-width="190">
          <template #default="scope">
            <div class="person-cell">
              <span class="person-mark">
                <i class="el-icon el-icon-user" aria-hidden="true"></i>
              </span>
              <span>
                <strong>{{ scope.row.userName || `用户 ${scope.row.userId}` }}</strong>
                <small>{{ scope.row.userId || shortId(scope.row.subjectId) }}</small>
              </span>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="来源" min-width="160">
          <template #default="scope">
            <div class="stacked-cell">
              <strong>{{ scope.row.ipAddress || '未知地址' }}</strong>
              <small>{{ authMethodText(scope.row.amr) }}</small>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="关联应用" min-width="210">
          <template #default="scope">
            <div class="tag-summary">
              <el-tag
                v-for="item in (scope.row.clientIds || []).slice(0, 2)"
                :key="item"
                size="small"
                effect="plain">{{ item }}</el-tag>
              <span class="muted-value" v-if="!(scope.row.clientIds || []).length">尚无关联应用</span>
              <span class="muted-value" v-else-if="scope.row.clientIds.length > 2">+{{ scope.row.clientIds.length - 2 }}</span>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="最近活动" width="170">
          <template #default="scope">
            {{ parseTime(scope.row.lastSeenAt) || '—' }}
          </template>
        </el-table-column>
        <el-table-column label="最晚结束时间" width="170">
          <template #default="scope">
            {{ parseTime(scope.row.absoluteExpiresAt) || '—' }}
          </template>
        </el-table-column>
        <el-table-column
          label="状态"
          width="105"
          align="center">
          <template #default="scope">
            <el-tag :type="statusType(scope.row.status)">
              {{ statusText(scope.row.status) }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="160"
          fixed="right"
          align="center">
          <template #default="scope">
            <div class="table-actions">
              <el-button
                type="text"
                icon="el-icon-view"
                @click="showDetail(scope.row)">查看</el-button>
              <el-button
                class="oauth-button-danger"
                v-if="scope.row.status !== 'revoked'"
                type="text"
                icon="el-icon-switch-button"
                v-hasPermi="['system:oauthSession:revoke']"
                @click="openRevoke(scope.row)">下线</el-button>
            </div>
          </template>
        </el-table-column>
        <template #empty>
          <el-empty description="没有符合条件的登录会话"/>
        </template>
      </el-table>
      <pagination
        v-show="total > 0"
        :page.sync="queryParams.pageNum"
        :limit.sync="queryParams.pageSize"
        :total="total"
        @pagination="getList"/>
    </PageFrame>

    <el-dialog
      custom-class="oauth-overlay"
      :visible.sync="revokeOpen"
      title="强制结束登录会话"
      width="min(540px, calc(100vw - 32px))"
      append-to-body
      :close-on-click-modal="false">
      <div class="impact-summary">
        <i class="el-icon el-icon-warning" aria-hidden="true"></i>
        <div>
          <strong>将结束 {{ revokeIds.length }} 个外部登录会话</strong>
          <p>
            将终止这些会话的登录状态及离线访问凭据。资源服务通过在线校验感知失效；应用本地登录状态由应用负责清理。
          </p>
        </div>
      </div>
      <label class="reason-label" for="session-revoke-reason">下线原因</label>
      <el-input
        id="session-revoke-reason"
        v-model="reason"
        type="textarea"
        :rows="3"
        maxlength="200"
        show-word-limit
        placeholder="说明为什么要强制下线，便于后续审计"/>
      <template #footer>
        <el-button @click="revokeOpen = false">取消</el-button>
        <el-button
          type="danger"
          :disabled="!reason.trim()"
          :loading="revoking"
          @click="revoke">确认下线</el-button>
      </template>
    </el-dialog>

    <el-drawer
      custom-class="oauth-overlay"
      :visible.sync="detailOpen"
      title="登录会话详情"
      size="min(640px, 92vw)">
      <div class="detail-hero">
        <span class="person-mark large">
          <i class="el-icon el-icon-user" aria-hidden="true"></i>
        </span>
        <div>
          <strong>{{ detail.userName || detail.userId || '用户' }}</strong>
          <p>
            {{ detail.ipAddress || '未知来源地址' }} ·
            {{ authMethodText(detail.amr) }}
          </p>
        </div>
        <el-tag :type="statusType(detail.status)">{{ statusText(detail.status) }}</el-tag>
      </div>
      <section class="detail-section">
        <h4>关联应用</h4>
        <div class="tag-summary">
          <el-tag
            v-for="item in detail.clientIds || []"
            :key="item"
            effect="plain">{{ item }}</el-tag>
          <span class="muted-value" v-if="!(detail.clientIds || []).length">尚无关联应用</span>
        </div>
      </section>
      <el-descriptions
        class="detail-descriptions"
        :column="1"
        border>
        <el-descriptions-item label="登录时间">{{
          parseTime(detail.authTime) || '—'
        }}</el-descriptions-item>
        <el-descriptions-item label="最近活动">{{
          parseTime(detail.lastSeenAt) || '—'
        }}</el-descriptions-item>
        <el-descriptions-item label="闲置后过期">{{
          parseTime(detail.idleExpiresAt) || '—'
        }}</el-descriptions-item>
        <el-descriptions-item label="最晚结束时间">{{
          parseTime(detail.absoluteExpiresAt) || '—'
        }}</el-descriptions-item>
        <el-descriptions-item label="下线原因">{{
          detail.revokeReason || '—'
        }}</el-descriptions-item>
      </el-descriptions>
      <el-collapse class="technical-details">
        <el-collapse-item title="查看技术标识" name="technical">
          <dl>
            <dt>会话 ID</dt>
            <dd>{{ detail.sid }}</dd>
            <dt>用户主体</dt>
            <dd>{{ detail.subjectId }}</dd>
            <dt>身份版本</dt>
            <dd>{{ detail.authVersion || '—' }}</dd>
            <dt>认证级别</dt>
            <dd>{{ detail.acr || '—' }}</dd>
          </dl>
        </el-collapse-item>
      </el-collapse>
    </el-drawer>
  </div>
</template>

<script>
/**
 * 组件名称：OAuthSession
 * 功能说明：登录会话管理页面，查看在线会话、来源与关联应用并支持强制下线
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 * @data {Array}   rows        - 会话列表数据
 * @data {Array}   selected    - 已选中的会话
 * @data {Number}  total       - 会话总条数
 * @data {Boolean} loading     - 列表加载状态
 * @data {Boolean} revoking    - 下线提交中状态
 * @data {Boolean} showSearch  - 是否显示搜索条件
 * @data {Boolean} revokeOpen  - 下线对话框是否可见
 * @data {Boolean} detailOpen  - 详情抽屉是否可见
 * @data {Array}   revokeIds   - 待下线的会话标识集合
 * @data {String}  reason      - 下线原因
 * @data {Object}  detail      - 当前会话详情
 * @data {Object}  queryParams - 查询参数
 *
 * 事件说明：
 * @event shortId        - 缩略显示会话标识
 * @event statusText     - 获取会话状态名称
 * @event statusType     - 获取会话状态标签类型
 * @event authMethodText - 获取认证方式的展示名称
 * @event getList        - 查询认证会话列表
 * @event handleQuery    - 搜索按钮操作
 * @event resetQuery     - 重置按钮操作
 * @event openRevoke     - 打开会话下线对话框
 * @event revoke         - 提交选中会话的下线原因
 * @event showDetail     - 查询并展示会话详情
 *
 * 插槽说明：
 */

/**
 * 2026/10/01     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'OAuthSession'
}
</script>

<script setup>
import { getCurrentInstance, reactive, ref } from 'vue'
import { getOAuthSession, listOAuthSessions, revokeOAuthSessions } from '@/api/system/oauthSession'
import PageFrame from '@/components/OAuthWorkspace/PageFrame.vue'

const { proxy } = getCurrentInstance()
const rows = ref([])
const selected = ref([])
const total = ref(0)
const loading = ref(false)
const revoking = ref(false)
const showSearch = ref(true)
const revokeOpen = ref(false)
const detailOpen = ref(false)
const revokeIds = ref([])
const reason = ref('')
const detail = ref({})
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  userId: undefined,
  ipAddress: undefined,
  status: 'active',
})

/**
 * 缩略显示会话标识
 * @param {String} value - 会话标识
 * @param {Number} size - 保留长度
 * @returns {String} 缩略后的标识
 */
function shortId(value, size = 12)
{
  if (!value) {
    return '—'
  }
  return value.length > size ? `${value.slice(0, size)}…` : value
}

/**
 * 获取会话状态名称
 * @param {String} value - 会话状态
 * @returns {String} 状态名称
 */
function statusText(value)
{
  return { active: '在线', revoked: '已下线', expired: '已过期' }[value] || '未知状态'
}

/**
 * 获取会话状态标签类型
 * @param {String} value - 会话状态
 * @returns {String} 状态标签类型
 */
function statusType(value)
{
  return { active: 'success', revoked: 'danger', expired: 'info' }[value] || 'info'
}

/**
 * 获取认证方式的展示名称
 * @param {Array} methods - 认证方式列表
 * @returns {String} 认证方式展示名称
 */
function authMethodText(methods = [])
{
  const labels = {
    pwd: '密码登录',
    password: '密码登录',
    otp: '动态验证码',
    mfa: '多重验证',
    sms: '短信验证',
  }
  return methods.length ? methods.map((item) => labels[item] || item).join(' + ') : '登录方式未知'
}

/**
 * 查询认证会话列表
 */
async function getList()
{
  loading.value = true
  try {
    const response = await listOAuthSessions(queryParams)
    selected.value = []
    rows.value = response.rows || []
    total.value = response.total || 0
  } finally {
    loading.value = false
  }
}

/**
 * 搜索按钮操作
 */
function handleQuery()
{
  queryParams.pageNum = 1
  getList()
}

/**
 * 重置按钮操作
 */
function resetQuery()
{
  proxy.resetForm('queryRef')
  handleQuery()
}

/**
 * 打开会话下线对话框
 * @param {Object} row - 会话行数据
 */
function openRevoke(row)
{
  revokeIds.value = row ? [row.sid] : selected.value.map((item) => item.sid)
  reason.value = ''
  revokeOpen.value = true
}

/**
 * 提交选中会话的下线原因
 */
async function revoke()
{
  revoking.value = true
  try {
    await revokeOAuthSessions(revokeIds.value.join(','), {
      reason: reason.value.trim(),
    })
    proxy.$modal.msgSuccess('会话已撤销')
    revokeOpen.value = false
    await getList()
  } finally {
    revoking.value = false
  }
}

/**
 * 查询并展示会话详情
 * @param {Object} row - 会话行数据
 */
async function showDetail(row)
{
  const response = await getOAuthSession(row.sid)
  detail.value = response.data
  detailOpen.value = true
}

getList()
</script>