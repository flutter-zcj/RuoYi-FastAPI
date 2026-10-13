<style lang="scss" scoped>
.drawer-wrap { padding: 0 20px 20px; }
.drawer-loading { display: flex; align-items: center; justify-content: center; gap: 8px; height: 120px; font-size: 13px; color: #aaa; }
.drawer-empty { padding: 60px 0; font-size: 13px; color: #bbb; text-align: center; }
.drawer-empty i { display: block; margin-bottom: 8px; font-size: 36px; }
.stat-row { margin-bottom: 16px; }
.stat-card { padding: 10px 14px; border: 1px solid #e8ecf0; background: #f7f9fb; text-align: center; border-radius: 6px; }
.stat-num { font-size: 22px; font-weight: 700; color: #2c3e50; }
.stat-num.success { color: #27ae60; }
.stat-num.danger { color: #e74c3c; }
.stat-label { margin-top: 4px; font-size: 11px; color: #95a5a6; }
.dict-item { display: grid; grid-template-columns: 1fr 1fr 1fr; overflow: hidden; margin-bottom: 8px; border: 1px solid #e8ecf0; border-radius: 6px; }
.dict-cell { display: grid; grid-template-columns: 70px 1fr; border-right: 1px solid #f0f4f8; }
.dict-cell:last-child { border-right: 0; }
.dict-cell-key { padding: 9px 14px; border-right: 1px solid #f0f4f8; background: #f7f9fb; font-size: 12px; color: #888; }
.dict-cell-val { display: flex; align-items: center; padding: 9px 14px; font-size: 13px; color: #2c3e50; word-break: break-all; }
</style>

<template>
  <el-drawer
    :title="drawerTitle"
    :visible.sync="localVisible"
    direction="rtl"
    size="700px"
    append-to-body>
    <div class="drawer-wrap">
      <div class="drawer-loading" v-if="loading">
        <i class="el-icon-loading"></i>
        <span>加载中...</span>
      </div>
      <div class="drawer-empty" v-else-if="!dataList.length">
        <i class="el-icon-document"></i>
        <div>暂无字典数据</div>
      </div>
      <template v-else>
        <el-row class="stat-row" :gutter="12">
          <el-col :span="disabledCount > 0 ? 8 : 12">
            <div class="stat-card">
              <div class="stat-num">{{ dataList.length }}</div>
              <div class="stat-label">共计条目</div>
            </div>
          </el-col>
          <el-col :span="disabledCount > 0 ? 8 : 12">
            <div class="stat-card">
              <div class="stat-num success">{{ normalCount }}</div>
              <div class="stat-label">正常</div>
            </div>
          </el-col>
          <el-col :span="8" v-if="disabledCount > 0">
            <div class="stat-card">
              <div class="stat-num danger">{{ disabledCount }}</div>
              <div class="stat-label">停用</div>
            </div>
          </el-col>
        </el-row>
        <div
          class="dict-item"
          v-for="item in dataList"
          :key="item.dictCode">
          <div class="dict-cell">
            <div class="dict-cell-key">标签</div>
            <div class="dict-cell-val">
              <el-tag
                v-if="item.listClass && item.listClass !== 'default'"
                :type="item.listClass === 'primary' ? '' : item.listClass"
                size="small">{{ item.dictLabel }}</el-tag>
              <span v-else>{{ item.dictLabel }}</span>
            </div>
          </div>
          <div class="dict-cell">
            <div class="dict-cell-key">键值</div>
            <div class="dict-cell-val">{{ item.dictValue }}</div>
          </div>
          <div class="dict-cell">
            <div class="dict-cell-key">状态</div>
            <div class="dict-cell-val">
              <el-tag :type="item.status === '0' ? 'success' : 'danger'" size="small">
                {{ item.status === '0' ? '正常' : '停用' }}
              </el-tag>
            </div>
          </div>
        </div>
      </template>
    </div>
  </el-drawer>
</template>

<script>
/**
 * 组件名称：detail
 * 功能说明：字典数据详情抽屉，展示指定字典类型下的全部字典数据
 * 研发时间：2026/07/31
 * 研发人员：insistence
 *
 * 参数说明：
 * @prop {Boolean} visible - 抽屉是否可见
 * @prop {Object}  row     - 当前字典类型行数据
 *
 * 变量说明：
 * @data {Boolean} localVisible - 内部抽屉可见状态
 * @data {Boolean} loading      - 数据加载状态
 * @data {Array}   dataList     - 字典数据列表
 *
 * 事件说明：
 * @event loadData - 加载字典数据列表
 *
 * 插槽说明：
 */

/**
 * 2026/07/31     insistence    初始创建
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import { listData } from '@/api/system/dict/data'

export default
{
  props:
  {
    visible: { type: Boolean, default: false },
    row: { type: Object, default: () => ({}) },
  },

  data()
  {
    return {
      localVisible: false,
      loading: false,
      dataList: [],
    }
  },

  computed:
  {
    /**
     * 抽屉标题，由字典名称与字典类型拼接
     * @returns {String} 抽屉标题
     */
    drawerTitle()
    {
      return this.row.dictName + '　' + (this.row.dictType || '')
    },

    /**
     * 状态正常的字典数据数量
     * @returns {Number} 正常条数
     */
    normalCount()
    {
      return this.dataList.filter((r) => r.status === '0').length
    },

    /**
     * 状态停用的字典数据数量
     * @returns {Number} 停用条数
     */
    disabledCount()
    {
      return this.dataList.filter((r) => r.status !== '0').length
    },
  },

  watch:
  {
    /**
     * 同步外部可见状态到内部状态
     * @param {Boolean} val - 外部可见状态
     */
    visible(val)
    {
      this.localVisible = val
    },

    /**
     * 内部可见状态变更时向上同步，并在打开时加载数据
     * @param {Boolean} val - 内部可见状态
     */
    localVisible(val)
    {
      this.$emit('update:visible', val)
      if (val) {
        this.loadData()
      } else {
        this.dataList = []
      }
    },
  },

  methods:
  {
    /**
     * 加载字典数据列表
     */
    loadData()
    {
      if (!this.row || !this.row.dictType) return
      this.loading = true
      this.dataList = []
      listData({ dictType: this.row.dictType, pageSize: 100, pageNum: 1 })
        .then((response) => {
          this.dataList = response.rows || []
        })
        .catch(() => {})
        .finally(() => {
          this.loading = false
        })
    },
  },
}
</script>