<style lang="scss" scoped>
.policy-note { margin-bottom: 20px; }
.filter-input { width: 180px; }
.policy-toolbar { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between; gap: 12px; margin-bottom: 16px; color: var(--oauth-text-color-regular); }
.policy-target strong, .policy-target small { display: block; overflow-wrap: anywhere; }
.policy-target strong { color: var(--oauth-text-color-primary); }
.policy-target small { margin-top: 4px; color: var(--oauth-text-color-secondary); }
</style>

<template>
  <div>
    <el-drawer
      custom-class="oauth-overlay"
      :visible.sync="visible"
      title="应用访问策略"
      size="min(960px, 96vw)"
      @open="handleQuery">
      <el-alert
        class="policy-note"
        title="可在用户首次授权前禁止访问应用。未配置策略时默认允许；解除禁止后，已撤销的授权不会恢复。"
        type="info"
        :closable="false"
        show-icon/>
      <el-form
        ref="queryRef"
        :model="queryParams"
        :inline="true"
        @submit.native.prevent="handleQuery">
        <el-form-item label="用户" prop="userId">
          <el-input
            class="filter-input"
            v-model="queryParams.userId"
            clearable
            placeholder="用户编号"/>
        </el-form-item>
        <el-form-item label="应用" prop="clientId">
          <el-input
            class="filter-input"
            v-model="queryParams.clientId"
            clearable
            placeholder="Client ID"/>
        </el-form-item>
        <el-form-item label="策略" prop="accessStatus">
          <el-select
            class="filter-input"
            v-model="queryParams.accessStatus"
            clearable
            placeholder="全部策略">
            <el-option label="禁止访问" value="blocked"/>
            <el-option label="允许访问" value="allowed"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            icon="el-icon-search"
            native-type="submit">查找</el-button>
          <el-button
            icon="el-icon-refresh"
            @click="resetQuery">清空</el-button>
        </el-form-item>
      </el-form>
      <div class="policy-toolbar">
        <span>已配置 {{ total }} 条策略</span>
        <el-button
          type="primary"
          icon="el-icon-plus"
          v-hasPermi="['system:oauthGrant:revoke']"
          @click="openAccess()">
          新增禁止策略
        </el-button>
      </div>
      <el-table v-loading="loading" :data="rows">
        <el-table-column label="用户" min-width="140">
          <template #default="scope">
            <div class="policy-target">
              <strong>{{ scope.row.userName }}</strong><small>{{ scope.row.userId }}</small>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="应用" min-width="160">
          <template #default="scope">
            <div class="policy-target">
              <strong>{{ scope.row.clientName }}</strong><small>{{ scope.row.clientId }}</small>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="策略" width="110">
          <template #default="scope">
            <el-tag :type="scope.row.accessStatus === 'blocked' ? 'danger' : 'success'">
              {{ scope.row.accessStatus === 'blocked' ? '禁止访问' : '允许访问' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="reason"
          label="操作原因"
          min-width="180"
          show-overflow-tooltip/>
        <el-table-column
          prop="updateBy"
          label="操作人"
          width="110"/>
        <el-table-column label="最近操作" width="170">
          <template #default="scope">{{ parseTime(scope.row.updateTime) || '—' }}</template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="110"
          fixed="right"
          align="center">
          <template #default="scope">
            <el-button
              type="text"
              :class="{ 'oauth-button-danger': scope.row.accessStatus !== 'blocked' }"
              v-hasPermi="['system:oauthGrant:revoke']"
              @click="openAccess(scope.row)">{{ scope.row.accessStatus === 'blocked' ? '解除禁止' : '禁止访问' }}</el-button>
          </template>
        </el-table-column>
        <template #empty><el-empty description="没有符合条件的访问策略"/></template>
      </el-table>
      <pagination
        v-show="total > 0"
        :page.sync="queryParams.pageNum"
        :limit.sync="queryParams.pageSize"
        :total="total"
        @pagination="getList"/>
      <el-dialog
        custom-class="oauth-overlay"
        :visible.sync="accessOpen"
        :title="accessBlocked ? '禁止用户访问应用' : '解除应用访问禁止'"
        width="min(540px, calc(100vw - 32px))"
        append-to-body
        :close-on-click-modal="false"
        :close-on-press-escape="!saving"
        :show-close="!saving">
        <el-alert
          class="policy-note"
          :title="
            accessBlocked
              ? '将撤销该用户对应用的全部现有授权，并阻止后续授权。'
              : '解除后允许重新授权，已有授权和令牌不会恢复。'
          "
          :type="accessBlocked ? 'warning' : 'info'"
          :closable="false"
          show-icon/>
        <el-form
          ref="accessRef"
          :model="accessForm"
          :rules="accessRules"
          label-position="top">
          <el-form-item label="用户编号" prop="userId">
            <el-input
              v-model="accessForm.userId"
              :disabled="!creating || saving"
              maxlength="19"
              inputmode="numeric"
              placeholder="输入用户管理中的用户编号"/>
          </el-form-item>
          <el-form-item label="Client ID" prop="clientId">
            <el-input
              v-model="accessForm.clientId"
              :disabled="!creating || saving"
              maxlength="128"
              placeholder="输入客户端管理中的 Client ID"/>
          </el-form-item>
          <el-form-item label="操作原因" prop="reason">
            <el-input
              v-model="accessForm.reason"
              :disabled="saving"
              type="textarea"
              :rows="3"
              maxlength="200"
              show-word-limit
              placeholder="填写原因，便于后续审计"/>
          </el-form-item>
        </el-form>
        <template #footer>
          <el-button
            :disabled="saving"
            @click="accessOpen = false">取消</el-button>
          <el-button
            :type="accessBlocked ? 'danger' : 'primary'"
            :loading="saving"
            @click="saveAccess">
            {{ accessBlocked ? '确认禁止' : '确认解除' }}
          </el-button>
        </template>
      </el-dialog>
    </el-drawer>
  </div>
</template>

<script>
/**
 * 组件名称：OAuthAccessPolicyDrawer
 * 功能说明：应用访问策略抽屉，查询用户对应用的独立访问策略并支持禁止或解除访问
 * 研发时间：2026/10/01
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} value - 抽屉是否显示
 *
 * 变量说明：
 * @data {Array}   rows          - 访问策略列表数据
 * @data {Number}  total         - 访问策略总条数
 * @data {Boolean} loading       - 列表加载状态
 * @data {Boolean} accessOpen    - 访问策略确认表单是否可见
 * @data {Boolean} accessBlocked - 当前是否进入禁止状态
 * @data {Boolean} creating      - 是否新增访问策略
 * @data {Boolean} saving        - 保存提交中状态
 * @data {Object}  queryParams   - 查询参数
 * @data {Object}  accessForm    - 访问策略表单数据
 * @data {Object}  accessRules   - 访问策略表单校验规则
 *
 * 事件说明：
 * @event input       - 更新抽屉显示状态
 * @event changed     - 访问策略变更后通知父组件刷新
 * @event getList     - 查询已配置的独立访问策略
 * @event handleQuery - 从第一页查询访问策略
 * @event resetQuery  - 清空查询条件
 * @event openAccess  - 打开新增或变更访问策略的确认表单
 * @event saveAccess  - 提交独立访问策略并同步授权列表
 *
 * 插槽说明：
 */

/**
 * 2026/10/01     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'OAuthAccessPolicyDrawer'
}
</script>

<script setup>
import { computed, getCurrentInstance, nextTick, reactive, ref } from 'vue'
import { listOAuthAccessPolicies, setOAuthClientAccess } from '@/api/system/oauthSession'

const props = defineProps({ value: { type: Boolean, default: false } })
const emit = defineEmits(['input', 'changed'])
const visible = computed({
  get: () => props.value,
  set: (value) => emit('input', value),
})
const { proxy } = getCurrentInstance()
const rows = ref([])
const total = ref(0)
const loading = ref(false)
const accessOpen = ref(false)
const accessBlocked = ref(true)
const creating = ref(false)
const saving = ref(false)
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  userId: undefined,
  clientId: undefined,
  accessStatus: undefined,
})
const accessForm = reactive({ userId: '', clientId: '', reason: '' })
const accessRules = {
  userId: [
    {
      required: true,
      pattern: /^[1-9]\d{0,18}$/,
      message: '请输入有效的用户编号',
      trigger: 'blur',
    },
  ],
  clientId: [{ required: true, whitespace: true, message: '请输入 Client ID', trigger: 'blur' }],
  reason: [{ required: true, whitespace: true, message: '请填写操作原因', trigger: 'blur' }],
}

/**
 * 查询已配置的独立访问策略
 */
async function getList()
{
  loading.value = true
  try {
    const response = await listOAuthAccessPolicies(queryParams)
    rows.value = response.rows || []
    total.value = response.total || 0
  } finally {
    loading.value = false
  }
}

/**
 * 从第一页查询访问策略
 */
function handleQuery()
{
  queryParams.pageNum = 1
  getList()
}

/**
 * 清空查询条件
 */
function resetQuery()
{
  proxy.resetForm('queryRef')
  handleQuery()
}

/**
 * 打开新增或变更访问策略的确认表单
 * @param {Object} row - 访问策略行数据
 */
function openAccess(row)
{
  creating.value = !row
  accessBlocked.value = !row || row.accessStatus !== 'blocked'
  Object.assign(accessForm, {
    userId: row ? String(row.userId) : '',
    clientId: row?.clientId || '',
    reason: '',
  })
  accessOpen.value = true
  nextTick(() => proxy.$refs.accessRef?.clearValidate())
}

/**
 * 提交独立访问策略并同步授权列表
 */
async function saveAccess()
{
  if (saving.value || !(await proxy.$refs.accessRef.validate().catch(() => false))) {
    return
  }
  saving.value = true
  try {
    await setOAuthClientAccess(accessForm.userId.trim(), accessForm.clientId.trim(), {
      blocked: accessBlocked.value,
      reason: accessForm.reason.trim(),
    })
    proxy.$modal.msgSuccess(accessBlocked.value ? '已禁止用户访问应用' : '已解除访问禁止')
    accessOpen.value = false
    emit('changed')
    await getList()
  } finally {
    saving.value = false
  }
}
</script>