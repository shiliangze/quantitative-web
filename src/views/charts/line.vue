<template>
  <el-form label-width="auto" style="max-width: 600px">
    <el-form-item>
      <el-button type="primary" @click="backtrace.dialog = true">回测</el-button>
      <el-button type="success">同步数据</el-button>
    </el-form-item>
  </el-form>
  <div class="flex gap-2">
    <el-tag type="primary" @click="findBalanceByPlanCode(1)">A股</el-tag>
    <el-tag type="success" @click="findBalanceByPlanCode(0)">美股</el-tag>
  </div>
  <el-table
    :data="balances"
    style="width: 100%">
    <el-table-column type="expand">
      <template v-slot="props">{{ props.row.stocks }}</template>
    </el-table-column>
    <el-table-column
      prop="name"
      label="名称"
      width="180">
    </el-table-column>
    <el-table-column
      prop="share"
      label="持仓预期"
      width="180">
    </el-table-column>
    <el-table-column
      prop="note"
      label="备注">
    </el-table-column>
  </el-table>
  <el-dialog v-model="backtrace.dialog" title="回测交易" width="500">
    <el-form :model="backtrace.form">
      <el-form-item label="方案类型">
        <el-radio-group v-model="backtrace.form.plan_code" fill="#409eff">
          <el-radio-button label="美股" :value="0" />
          <el-radio-button label="A股" :value="1" />
        </el-radio-group>
      </el-form-item>
      <el-form-item label="交易类型">
        <el-radio-group v-model="backtrace.form.trade_type" fill="#409eff">
          <el-radio-button label="历史交易" :value="0" />
          <el-radio-button label="回测交易" :value="1" />
        </el-radio-group>
      </el-form-item>
      <el-form-item label="起始日类型">
        <el-radio-group v-model="backtrace.form.start_type" fill="#409eff">
          <el-radio-button label="IPO" :value="0" />
          <el-radio-button label="首次交易" :value="1" />
        </el-radio-group>
      </el-form-item>
      <el-form-item label="启动资金">
        <el-input-number v-model="backtrace.form.cash" :step="10000" :max="1000000" />
      </el-form-item>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="backtrace.dialog = false">Cancel</el-button>
        <el-button type="primary" @click="backTrace">
          执行回测
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import request from '@/utils/request';
import { el } from 'element-plus/es/locales.mjs';
// import Chart from '@/componeents/Charts/KLine.vue';

interface Stock {
  id?: number;
  code: string;
  name: string;
  note?: string;
  // 其他股票相关的属性
}

interface Balance {
  id: number;
  name: string;
  share: string;
  note?: string;
  stocks: Stock[]; // 预定义 stocks 属性
}

export default defineComponent({
  name: 'LineChart',
  data() {
    return {
      balances: [] as Balance[],
      backtrace: {
        dialog: false,
        form: {
          plan_code: 1,
          trade_type: 1,
          start_type: 0,
          cash: 200000
        }
      },
      loading: false
    };
  },
  methods: {
    findBalanceByPlanCode(planCode) {
      this.loading = true;
      // 硬编码股票代码为"512170.SHH"
      request({
        url: `/balance/find_by_plan/${planCode}`,
        method: 'get'
      }).then(response => {
        this.balances = response.data || [];
      }).finally(() => {
        this.loading = false;
      });
    },
    backTrace() {
      this.loading = true;
      // 硬编码股票代码为"512170.SHH"
      request({
        url: `/trader/back_track`,
        method: 'post',
        data: this.backtrace.form
      }).then(response => {
        // 处理响应数据
        console.log(response.data);
      }).finally(() => {
        this.backtrace.dialog = false;
        this.loading = false;
      });
    }
  }
});
</script>

<style scoped>
.chart-container{
  position: relative;
  width: 100%;
 height: calc(100vh - 500px);
}
</style>
