<template>
  <div>
    <div class="pie">
      <div id="pie1">
        <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
        <div id="main1" style="float:left;width:50%;height: 400px"></div>
      </div>
      <div id="pie2">
        <div id="main2" style="float:right;width:50%;height: 400px"></div>
      </div>
    </div>
  </div>
</template>

<script>// 引入基本模板
let echarts = require('echarts/lib/echarts');
import {deleteAction, getAction, postAction} from '@/api/manage';
// 引入饼状图组件
require('echarts/lib/chart/pie')
// 引入提示框和title组件
require('echarts/lib/component/tooltip')
require('echarts/lib/component/title')


export default {
  data(){
    return {
      url: {
        list: "/renche/projectItem/qryStatus",
        list1: "/renche/projectItem/qryStatus1",
      }
    }
  },


  created(){


  },
  mounted(){
    this.getQueryStatus();
    this.getQueryStatus1();
    this.initData();
  },
  methods:{
    getQueryStatus(){
      let that = this;
      getAction(this.url.list).then((res) => {
        if (res.success) {
          // 基于准备好的dom，初始化echarts实例
          var myChart = echarts.init(document.getElementById('main1'));
          // 绘制图表
          myChart.setOption({
            title : {
              text: '人员车辆工程点统计',//主标题
              subtext: '进度统计',//副标题
              x:'center',//x轴方向对齐方式
            },
            tooltip : {
              trigger: 'item',
              formatter: "{a} <br/>{b} : {c} ({d}%)"
            },
            legend: {
              orient: 'vertical',
              bottom: 'bottom',
              data: ['未开始','进行中','已完成']
            },
            series : [
              {
                name: '工程点数量',
                type: 'pie',
                radius : '55%',
                center: ['50%', '60%'],
                data:res.result,
                /*[
                  {value:335, name:'未开始'},
                  {value:310, name:'进行中'},
                  {value:234, name:'已完成'}

                ],*/
                itemStyle: {
                  emphasis: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                  }
                }
              }
            ]
          },true);

        }
      });
    },
    getQueryStatus1(){
      let that = this;
      getAction(this.url.list1).then((res) => {
        if (res.success) {
          // 基于准备好的dom，初始化echarts实例
          var myChart = echarts.init(document.getElementById('main2'));
          // 绘制图表
          myChart.setOption({
            title : {
              text: '4G视频工程点统计',//主标题
              subtext: '进度统计',//副标题
              x:'center',//x轴方向对齐方式
            },
            tooltip : {
              trigger: 'item',
              formatter: "{a} <br/>{b} : {c} ({d}%)"
            },
            legend: {
              orient: 'vertical',
              bottom: 'bottom',
              data: ['未开始','进行中','已完成']
            },
            series : [
              {
                name: '工程点数量',
                type: 'pie',
                radius : '55%',
                center: ['50%', '60%'],
                data:res.result,
                /*[
                  {value:335, name:'未开始'},
                  {value:310, name:'进行中'},
                  {value:234, name:'已完成'}

                ],*/
                itemStyle: {
                  emphasis: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                  }
                }
              }
            ]
          },true);

        }
      });
    },
    //初始化数据
    initData() {


    }
  }
}
</script>
