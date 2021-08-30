<template>
  <div>
     <!-- 面包屑导航区域 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>数据统计</el-breadcrumb-item>
  <el-breadcrumb-item>数据报表</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片视图 -->
<el-card>
 <!-- prepare a DOM container with width and height -->
    <div id="main" style="width: 700px;height:400px;"></div>
</el-card>
  </div>
</template>

<script>
//1.导入echarts
import *as echarts from 'echarts'
import axios from 'axios';
export default {
  data() {
    return{
      //需要合并的数据
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created() {

  },
  //此时页面上的元素已经被渲染完毕
  mounted() {
     // based on prepared DOM, initialize echarts instance
        var myChart = echarts.init(document.getElementById('main'));

        axios.get('https://lianghj.top:8888/api/private/v1/reports/type/1').then(res => {
          if(res.data.meta.status !== 200) {
            return this.$message.error('获取折线图数据失败~')
          }
          //展示数据
          var a = JSON.parse(JSON.stringify(this.options))
          var b = JSON.parse(JSON.stringify(res.data.data))
          var result = {...a,...b}
          myChart.setOption(result);
        })

        //准备数据和配置项
        //  var option = {
        //     title: {
        //         text: 'ECharts entry example'
        //     },
        //     tooltip: {},
        //     legend: {
        //         data:['Sales']
        //     },
        //     xAxis: {
        //         data: ["shirt","cardign","chiffon shirt","pants","heels","socks"]
        //     },
        //     yAxis: {},
        //     series: [{
        //         name: 'Sales',
        //         type: 'bar',
        //         data: [5, 20, 36, 10, 10, 20]
        //     }]
        // };
     
  },
  methods: {

  }
}
</script>

<style>

</style>