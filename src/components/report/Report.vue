<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 为Echarts 准备一个DOM -->
      <div id="main" style="width:750px;height:400px"></div>
    </el-card>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  name: '',
  data() {
    return {
      // 需要合并的数据
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
  computed: {},
  // 页面元素已经被挂载完毕
  async mounted() {
    // 基于准备好的 DOM ， 初始化 echarts 实例
    var myChart = echarts.init(document.getElementById('main'))

    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) {
      return this.$message.error('获取折线图数据失败')
    }

    // 合并数据和配置项
    const result = Object.assign(this.options, res.data)
    result.xAxis[0].boundaryGap = false
    myChart.setOption(result)
  },
  watch: {},
  methods: {}
}
</script>

<style scoped></style>
