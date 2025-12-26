<template>
  <div class="flex gap-2">
    <el-tag type="primary" @click="test(1)">A股</el-tag>
    <el-tag type="success" @click="test(0)">美股</el-tag>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import request from '@/utils/request';
// import Chart from '@/componeents/Charts/KLine.vue';

export default defineComponent({
  name: 'LineChart',
  methods: {
    test(investCode) {
      this.loading = true;
      // 硬编码股票代码为"512170.SHH"
      request({
        url: `/balance/find_by_invest/${investCode}`,
        method: 'get'
      }).then(response => {
        // 处理响应数据
        console.log(response.data);
      }).finally(() => {
        console.log('AAA');
      });
    }
    // test(investCode) {
    //   this.loading = true;
    //   // 硬编码股票代码为"512170.SHH"
    //   request({
    //     url: `/stock/balance/${investCode}`,
    //     method: 'post',
    //     data: { 'investCode': '1' }
    //   }).then(response => {
    //     // 处理响应数据
    //     this.klineData = response.data || [];
    //     this.initChart();
    //   }).finally(() => {
    //     this.loading = false;
    //   });
    // }
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