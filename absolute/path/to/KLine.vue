mounted() {
  getKLine('512170.SHH').then(data => {
    console.log('原始数据:', data); // 调试日志，查看返回的数据结构
    
    // 确保data是数组
    if (!Array.isArray(data)) {
      console.error('数据格式错误：不是数组', data);
      return;
    }
    
    // 检查第一个元素是否为对象格式
    if (data.length > 0 && typeof data[0] === 'object' && data[0] !== null) {
      // 数据格式转换：将对象数组转换为组件期望的数组格式
      this.kLineData = data.map(item => [
        item.date,   // 日期
        item.open,   // 开盘价
        item.close,  // 收盘价
        item.low,    // 最低价
        item.high,   // 最高价
        item.volume  // 成交量
      ]);
    } else {
      // 如果已经是数组格式，直接使用
      this.kLineData = data;
    }
    
    console.log('转换后的数据:', this.kLineData); // 调试日志，查看转换后的数据
    
    // 限制数据量，避免图表渲染过多数据
    const maxDataCount = 100; // 最多显示100条数据
    if (this.kLineData.length > maxDataCount) {
      this.kLineData = this.kLineData.slice(-maxDataCount);
    }
    
    // 计算技术指标
    this.calculateBOLL();
    this.calculateBBI();
    
    // 初始化图表
    this.initChart();
  }).catch(error => {
    console.error('获取K线数据失败:', error);
    // 可选：在请求失败时使用模拟数据
    // this.useMockData();
  });
}