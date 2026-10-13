<style lang="scss" scoped></style>

<template>
  <el-dialog
    title="操作日志详细"
    :visible.sync="visible"
    width="780px"
    append-to-body
    @close="$emit('update:visible', false)">
    <div class="detail-wrap">
      <!-- 基本信息 -->
      <div class="detail-card">
        <div class="detail-card-title"><i class="el-icon-info"></i> 基本信息</div>
        <el-row class="detail-row">
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">操作模块</span><span class="detail-value">{{ form.title }}</span>
            </div>
          </el-col>
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">业务类型</span><span class="detail-value">{{ typeLabel }}</span>
            </div>
          </el-col>
        </el-row>
        <el-row class="detail-row">
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">操作时间</span><span class="detail-value">{{ parseTime(form.operTime) || '-' }}</span>
            </div>
          </el-col>
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">执行状态</span>
              <el-tag
                v-if="form.status === 0"
                type="success"
                size="small"><i class="el-icon-check"></i> 正常</el-tag>
              <el-tag
                v-else
                type="danger"
                size="small"><i class="el-icon-close"></i> 异常</el-tag>
            </div>
          </el-col>
        </el-row>
      </div>

      <!-- 操作人员 -->
      <div class="detail-card">
        <div class="detail-card-title"><i class="el-icon-user"></i> 操作人员</div>
        <el-row class="detail-row">
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">操作人员</span><span class="detail-value">{{ form.operName }}</span>
            </div>
          </el-col>
          <el-col :span="12" v-if="form.deptName">
            <div class="detail-item">
              <span class="detail-label">所属部门</span><span class="detail-value">{{ form.deptName }}</span>
            </div>
          </el-col>
        </el-row>
        <el-row class="detail-row">
          <el-col :span="24">
            <div class="detail-item">
              <span class="detail-label">操作地址</span>
              <span class="detail-value">{{ form.operIp }}&nbsp;&nbsp;<span class="detail-location">{{ form.operLocation }}</span></span>
            </div>
          </el-col>
        </el-row>
      </div>

      <!-- 请求信息 -->
      <div class="detail-card">
        <div class="detail-card-title"><i class="el-icon-sort"></i> 请求信息</div>
        <el-row class="detail-row">
          <el-col :span="24">
            <div class="detail-item">
              <span class="detail-label">请求地址</span>
              <span class="detail-value">
                <span :class="'method-tag method-' + form.requestMethod">{{ form.requestMethod }}</span>
                {{ form.operUrl }}
              </span>
            </div>
          </el-col>
        </el-row>
        <el-row class="detail-row">
          <el-col :span="24">
            <div class="detail-item">
              <span class="detail-label">操作方法</span><span class="detail-value mono">{{ form.method }}</span>
            </div>
          </el-col>
        </el-row>
        <el-row class="detail-row">
          <el-col :span="12">
            <div class="detail-item">
              <span class="detail-label">消耗时间</span><span class="detail-value">{{ form.costTime }} 毫秒</span>
            </div>
          </el-col>
        </el-row>
      </div>

      <!-- 请求参数 -->
      <div class="detail-card">
        <div class="detail-card-title"><i class="el-icon-upload2"></i> 请求参数</div>
        <div class="code-body">
          <div class="code-wrap">
            <div class="code-action">
              <el-button
                size="mini"
                icon="el-icon-copy-document"
                @click="copyText(form.operParam)">复制</el-button>
            </div>
            <pre class="code-pre">{{ formatJson(form.operParam) }}</pre>
          </div>
        </div>
      </div>

      <!-- 返回参数 -->
      <div class="detail-card">
        <div class="detail-card-title"><i class="el-icon-download"></i> 返回参数</div>
        <div class="code-body">
          <div class="code-wrap">
            <div class="code-action">
              <el-button
                size="mini"
                icon="el-icon-copy-document"
                @click="copyText(form.jsonResult)">复制</el-button>
            </div>
            <pre class="code-pre">{{ formatJson(form.jsonResult) }}</pre>
          </div>
        </div>
      </div>

      <!-- 异常信息 -->
      <div class="detail-card" v-if="form.status !== 0">
        <div class="detail-card-title error-title"><i class="el-icon-warning"></i> 异常信息</div>
        <div class="error-body">
          <div class="error-msg">{{ form.errorMsg }}</div>
        </div>
      </div>
    </div>
  </el-dialog>
</template>

<script>
/**
 * 组件名称：OperlogDetail
 * 功能说明：操作日志详情弹窗，展示基本信息、操作人员、请求信息与请求/返回参数
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} visible - 弹窗是否可见
 * @prop {Object}  row     - 当前操作日志行数据
 *
 * 变量说明：
 *
 * 事件说明：
 * @event update:visible - 关闭弹窗时向上同步可见状态
 *
 * 插槽说明：
 */

/**
 * 2026/07/31     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

export default
{
  name: 'OperlogDetail',

  dicts: ['sys_oper_type'],

  props:
  {
    visible: { type: Boolean, default: false },
    row: { type: Object, default: () => ({}) },
  },

  computed:
  {
    /**
     * 当前行数据
     * @returns {Object} 操作日志行数据
     */
    form()
    {
      return this.row || {}
    },

    /**
     * 业务类型名称
     * @returns {String} 业务类型标签
     */
    typeLabel()
    {
      return this.selectDictLabel(this.dict.type.sys_oper_type, this.form.businessType) || '-'
    },
  },

  methods:
  {
    /**
     * 格式化 JSON 字符串
     * @param {String} str - JSON 字符串
     * @returns {String} 格式化后的字符串
     */
    formatJson(str)
    {
      if (!str) return '（无数据）'
      try
      {
        return JSON.stringify(JSON.parse(str), null, 2)
      } catch
      {
        return str
      }
    },

    /**
     * 复制文本到剪贴板
     * @param {String} str - 待复制内容
     */
    copyText(str)
    {
      const text = this.formatJson(str)
      if (navigator.clipboard)
      {
        navigator.clipboard
          .writeText(text)
          .then(() => this.$message({ message: '已复制', type: 'success', duration: 1500 }))
      } else
      {
        const ta = document.createElement('textarea')
        ta.value = text
        document.body.appendChild(ta)
        ta.select()
        document.execCommand('copy')
        document.body.removeChild(ta)
        this.$message({ message: '已复制', type: 'success', duration: 1500 })
      }
    },
  },
}
</script>