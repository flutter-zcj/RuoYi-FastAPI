<style lang="scss" scoped>
.panel-group { margin-top: 18px;
  .card-panel-col { margin-bottom: 32px; }
  .card-panel { height: 108px; cursor: pointer; font-size: 12px; position: relative; overflow: hidden; color: #666; background: #fff; box-shadow: 4px 4px 40px rgba(0, 0, 0, 0.05); border-color: rgba(0, 0, 0, 0.05);
    &:hover {
      .card-panel-icon-wrapper { color: #fff; }
      .icon-people { background: #40c9c6; }
      .icon-message { background: #36a3f7; }
      .icon-money { background: #f4516c; }
      .icon-shopping { background: #34bfa3; }
    }
    .icon-people { color: #40c9c6; }
    .icon-message { color: #36a3f7; }
    .icon-money { color: #f4516c; }
    .icon-shopping { color: #34bfa3; }
    .card-panel-icon-wrapper { float: left; margin: 14px 0 0 14px; padding: 16px; transition: all 0.38s ease-out; border-radius: 6px; }
    .card-panel-icon { float: left; font-size: 48px; }
    .card-panel-description { float: right; font-weight: bold; margin: 26px; margin-left: 0px;
      .card-panel-text { line-height: 18px; color: rgba(0, 0, 0, 0.45); font-size: 16px; margin-bottom: 12px; }
      .card-panel-num { font-size: 20px; }
    }
  }
}
@media (max-width: 550px) {
  .card-panel-description { display: none; }
  .card-panel-icon-wrapper { float: none !important; width: 100%; height: 100%; margin: 0 !important;
    .svg-icon { display: block; margin: 14px auto !important; float: none !important; }
  }
}
</style>

<template>
  <el-row
    class="panel-group"
    :gutter="40">
    <el-col
      class="card-panel-col"
      :xs="12"
      :sm="12"
      :lg="6">
      <div
        class="card-panel"
        @click="handleSetLineChartData('newVisitis')">
        <div class="card-panel-icon-wrapper icon-people">
          <svg-icon icon-class="peoples" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">访客</div>
          <count-to
            class="card-panel-num"
            :start-val="0"
            :end-val="102400"
            :duration="2600"/>
        </div>
      </div>
    </el-col>
    <el-col
      class="card-panel-col"
      :xs="12"
      :sm="12"
      :lg="6">
      <div
        class="card-panel"
        @click="handleSetLineChartData('messages')">
        <div class="card-panel-icon-wrapper icon-message">
          <svg-icon icon-class="message" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">消息</div>
          <count-to
            class="card-panel-num"
            :start-val="0"
            :end-val="81212"
            :duration="3000"/>
        </div>
      </div>
    </el-col>
    <el-col
      class="card-panel-col"
      :xs="12"
      :sm="12"
      :lg="6">
      <div
        class="card-panel"
        @click="handleSetLineChartData('purchases')">
        <div class="card-panel-icon-wrapper icon-money">
          <svg-icon icon-class="money" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">金额</div>
          <count-to
            class="card-panel-num"
            :start-val="0"
            :end-val="9280"
            :duration="3200"/>
        </div>
      </div>
    </el-col>
    <el-col
      class="card-panel-col"
      :xs="12"
      :sm="12"
      :lg="6">
      <div
        class="card-panel"
        @click="handleSetLineChartData('shoppings')">
        <div class="card-panel-icon-wrapper icon-shopping">
          <svg-icon icon-class="shopping" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">订单</div>
          <count-to
            class="card-panel-num"
            :start-val="0"
            :end-val="13600"
            :duration="3600"/>
        </div>
      </div>
    </el-col>
  </el-row>
</template>

<script>
/**
 * 组件名称：PanelGroup
 * 功能说明：首页顶部统计面板组件，展示访客、消息、金额、订单四项数据
 * 研发时间：2024/01/22
 * 研发人员：insistence
 *
 * 参数说明：
 *
 * 变量说明：
 *
 * 事件说明：
 * @event handleSetLineChartData - 切换折线图数据类型
 *
 * 插槽说明：
 */

/**
 * 2026/10/13     Trae          按编码规范调整格式与注释
 */

import CountTo from 'vue-count-to'

export default
{
  components:
  {
    CountTo,
  },

  methods:
  {
    /**
     * 触发顶部统计卡片切换折线图数据事件
     * @param {String} type - 数据类型
     */
    handleSetLineChartData(type)
    {
      this.$emit('handleSetLineChartData', type)
    },
  },
}
</script>