<template>
  <el-form label-width="auto" style="max-width: 600px">
    <el-form-item>
      <el-button type="primary" @click="backtrace.dialog = true">回测</el-button>
      <el-button type="success">同步数据</el-button>
    </el-form-item>
  </el-form>
  <el-table
    :data="backtrace.response.balances"
    style="width: 100%">
    <el-table-column type="expand">
      <template v-slot="props">
        <el-table
          :data="props.row.stocks"
          style="width: 100%">
          <el-table-column
            prop="date"
            label="交易日期"
            width="100">
          </el-table-column>
          <el-table-column
            prop="stock_name"
            label="名称"
            width="150">
          </el-table-column>
          <el-table-column
            prop="direction"
            label="交易方向"
            :formatter="formatTradeDirection"
            width="100">
          </el-table-column>
          <el-table-column
            prop="trade_price"
            label="交易价格"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="trade_quantity"
            label="交易数量"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="close"
            label="收盘价"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="quantity"
            label="持仓数量"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="amount"
            label="市值"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="put"
            label="卖出指导价"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="call"
            label="买入指导价"
            :formatter="formatMoney"
            width="100"> </el-table-column>
          <el-table-column
            prop="profit"
            label="利润"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
          <el-table-column
            prop="profit_rate"
            label="年化收益"
            :formatter="formatMoney"
            width="100">
          </el-table-column>
        </el-table>
      </template>
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
import type { TableColumnCtx } from 'element-plus';

interface Balance {
  id: number;
  name: string;
  share: string;
  note?: string;
  stocks: BackTrackStock[]; // 预定义 stocks 属性
}

interface BackTrackStock {
  date: Date;
  stock_id: number;
  stock_name: string;
  balance_id: number;
  direction: number;
  trade_price: number;
  trade_quantity: number;
  close: number;
  quantity: number;
  amount: number;
  put: number;
  call: number;
  profit: number;
  profit_rate: number;
}
interface BackTraceResponse {
  balances: Balance[];
  back_track_stocks: BackTrackStock[];
}
// 定义行数据类型
interface RowData {
  row: number | string
  // 可添加其他字段，如 id: number, name: string 等
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
        },
        response: {} as BackTraceResponse,
        tableData: [] as BackTrackStock[],
        balances: [] as Balance[]
      },
      loading: false
    };
  },
  methods: {
    backTrace() {
      this.loading = true;
      // 硬编码股票代码为"512170.SHH"
      request({
        url: `/trader/back_track`,
        method: 'post',
        data: this.backtrace.form
      }).then(response => {
        // 处理响应数据
        this.backtrace.response = response.data;
        // this.backtrace.tableData = response.data.back_track_stocks.filter(
        //   (item: BackTrackStock) => item.direction !== 0
        // );
        this.backtrace.balances = response.data.balances;
        this.backtrace.balances.forEach(balance => {
          balance.stocks = response.data.back_track_stocks.filter(
            (item: BackTrackStock) => item.balance_id === balance.id && item.direction !== 0
          );
        });
      }).finally(() => {
        this.backtrace.dialog = false;
        this.loading = false;
      });
    },
    formatMoney(
      row: RowData,
      column: TableColumnCtx<RowData>,
      cellValue: string | number,
      index: number
    ): string {
      const num = parseFloat(cellValue as string);
      return isNaN(num) ? '0.000' : num.toFixed(3);
    },
    formatTradeDirection(row: RowData, column: TableColumnCtx<RowData>, cellValue: string | number) {
      const directionMap = {
        '-1': '卖出',
        '0': '无交易',
        '1': '买入'
      };
      return directionMap[cellValue] || '未知';
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
