<template>
  <div :id="id" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import { defineComponent } from 'vue';
import * as echarts from 'echarts';
import resize from './mixins/resize';
import request from '@/utils/request';
import { debounce } from '@/utils/index';

export default defineComponent({
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    id: {
      type: String,
      default: 'kline-chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '400px'
    }
  },
  data() {
    return {
      chart: null,
      klineData: [],
      loading: false
    };
  },
  mounted() {
  },
  beforeUnmount() {

  },
  methods: {
    // 获取K线数据
    fetchKLineData() {
      this.loading = true;
      // 硬编码股票代码为"512170.SHH"
      request({
        url: '/stock/kline',
        method: 'post',
        data: { 'ticker': '512170.SHH' }
      }).then(response => {
        // 处理响应数据
        this.klineData = response.data || [];
        this.initChart();
      }).finally(() => {
        this.loading = false;
      });
    },

    // 初始化图表
    initChart() {
      if (!this.chart) {
        this.chart = echarts.init(document.getElementById(this.id));
      }

      const { dates, klineValues, volumes } = this.processKLineData();

      const option = {
        backgroundColor: '#fff',
        title: {
          text: '股票K线图',
          left: 'center',
          textStyle: {
            color: '#333'
          }
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          backgroundColor: 'rgba(245, 245, 245, 0.8)',
          borderWidth: 1,
          borderColor: '#ccc',
          textStyle: {
            color: '#000'
          }
        },
        legend: {
          data: ['K线', '成交量'],
          top: 30
        },
        grid: [
          {
            left: '3%',
            right: '4%',
            top: '15%',
            height: '60%'
          },
          {
            left: '3%',
            right: '4%',
            top: '80%',
            height: '15%'
          }
        ],
        xAxis: [
          {
            type: 'category',
            data: dates,
            scale: true,
            boundaryGap: false,
            axisLine: { lineStyle: { color: '#333' } },
            axisTick: { show: false },
            splitLine: { show: false },
            axisLabel: {
              color: '#666',
              interval: Math.floor(dates.length / 10)
            }
          },
          {
            type: 'category',
            gridIndex: 1,
            data: dates,
            scale: true,
            boundaryGap: false,
            axisLine: { show: false },
            axisTick: { show: false },
            splitLine: { show: false },
            axisLabel: { show: false }
          }
        ],
        yAxis: [
          {
            scale: true,
            axisLine: { show: false },
            axisTick: { show: false },
            splitLine: { lineStyle: { color: '#eee' } },
            axisLabel: { color: '#666' }
          },
          {
            scale: true,
            gridIndex: 1,
            splitNumber: 2,
            axisLabel: { show: false },
            axisLine: { show: false },
            axisTick: { show: false },
            splitLine: { show: false }
          }
        ],
        dataZoom: [
          {
            type: 'inside',
            start: 50,
            end: 100
          },
          {
            show: true,
            type: 'slider',
            top: '90%',
            start: 50,
            end: 100
          }
        ],
        series: [
          {
            name: 'K线',
            type: 'candlestick',
            data: klineValues,
            itemStyle: {
              color: '#ef232a',
              color0: '#14b143',
              borderColor: '#ef232a',
              borderColor0: '#14b143'
            },
            emphasis: {
              itemStyle: {
                color: '#ff7f50',
                color0: '#87ceeb',
                borderColor: '#ff7f50',
                borderColor0: '#87ceeb'
              }
            },
            large: true
          },
          {
            name: '成交量',
            type: 'bar',
            xAxisIndex: 1,
            yAxisIndex: 1,
            data: volumes,
            itemStyle: {
              color: function(params) {
                const dataOhlc = klineValues[params.dataIndex];
                return dataOhlc[1] > dataOhlc[4] ? '#ef232a' : '#14b143';
              }
            }
          }
        ]
      };

      this.chart.setOption(option);
    },

    // 处理K线数据
    processKLineData() {
      const dates = [];
      const klineValues = [];
      const volumes = [];

      if (!this.klineData || !this.klineData.length) {
        return { dates, klineValues, volumes };
      }

      // 按日期正序排序
      const sortedData = [...this.klineData].sort((a, b) => new Date(a.date) - new Date(b.date));

      sortedData.forEach(item => {
        dates.push(item.date);
        // K线数据格式: [open, close, low, high]
        klineValues.push([item.open, item.close, item.low, item.high]);
        // 成交量数据
        volumes.push(item.volume);
      });

      return { dates, klineValues, volumes };
    }
  }
});
</script>

<style scoped>
.chart {
  width: 100%;
  height: 100%;
}
</style>
