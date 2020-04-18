<template>
  <div class="page-header-index-wide">

    <a-row :gutter="24">
      <a-col :sm="24" :md="400" :xl="10" :style="{ marginBottom: '200px' }">
        <div id="main1" style="float:left;width:60%;height: 400px"></div>
      </a-col>
      <a-col :sm="24" :md="400" :xl="10" :style="{ marginBottom: '200px' }">
        <div id="main2" style="float:right;width:60%;height: 400px"></div>

      </a-col>

    </a-row>

    <a-card :loading="loading" :bordered="false" :body-style="{padding: '0'}">
      <div class="salesCard">
        <a-tabs default-active-key="1" size="large" :tab-bar-style="{marginBottom: '24px', paddingLeft: '16px'}">
          <div class="extra-wrapper" slot="tabBarExtraContent">
            <div class="extra-item">
              <a @click="showAllMessage">查看全部</a>
            </div>
          </div>
          <a-tab-pane loading="true" tab="消息提醒" key="1" style="padding-left: 16px;">
            <a-row>
              <a-col :xl="16" :lg="12" :md="12" :sm="24" :xs="24">
                <a-list>

                  <a-list-item :key="index" v-for="(item, index) in dataSource">

                    <a-list-item-meta>
                      <a slot="title" @click="showDetail(item)">
                        <span style="padding-right: 8px">{{index+1}}.</span>{{ item.messageContent }}
                      </a>
                      <div slot="description" >{{ item.createTime }}</div>
                      <a-avatar slot="avatar" style="background-color: #78b1f8;"><a-icon type="sound" /></a-avatar>
                    </a-list-item-meta>
                  </a-list-item>
                </a-list>
              </a-col>
            </a-row>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-card>

    <!-- 表单区域 -->
    <contract-info-modal ref="contractInfoModal" @ok="modalFormOk"></contract-info-modal>

    <message-show ref="messageShow" @close="modalShowOk"></message-show>
  </div>
</template>

<script>

  import ACol from "ant-design-vue/es/grid/Col"
  import ATooltip from "ant-design-vue/es/tooltip/Tooltip"

  import {getLoginfo} from '@/api/api.js'
  import { getAction, httpAction} from '@/api/manage'
  import ContractInfoModal from '../work/project/modules/ContractInfoModal'
  import MessageShow from '../work/message/MessageShow'

  let echarts = require('echarts/lib/echarts');
  import {postAction} from '@/api/manage';
  // 引入饼状图组件
  require('echarts/lib/chart/pie')
  // 引入提示框和title组件
  require('echarts/lib/component/tooltip')
  require('echarts/lib/component/title')
  require('echarts/lib/component/legend')



  const rankList = []



  export default {
    name: "Analysis",
    components: {
      ATooltip,
      ACol,

      ContractInfoModal,
      MessageShow
    },
    data() {
      return {
        loading: true,
        rankList,
        loginfo:{},
        user: {},
        dataSource: [],
        record:{},
        url: {
          list: "/renche/messageInfo/notReadlist",
          edit: "/renche/messageInfo/edit",
          contractList: "/renche/contractInfo/list",
          filelist: "/renche/purchase/fileList",
          list2: "/renche/projectItem/qryStatus",
          list1: "/renche/projectItem/qryStatus1",
        },
      }
    },
    computed: {
      userInfo() {
        return this.$store.getters.userInfo;
      }
    },
    created() {
      setTimeout(() => {
        this.loading = !this.loading
      }, 1000)
      this.initLogInfo();
      this.loadData();
    },
    mounted(){
      this.getQueryStatus();
      this.getQueryStatus1();

    },
    methods: {

      initLogInfo () {
        getLoginfo(null).then((res)=>{
          if(res.success){
            this.loginfo = res.result;
          }
        })
      },
      loadData(arg){
        this.user = this.userInfo;
        getAction(this.url.list, {sysId: this.user.id, pageSize: 5 }).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
          }
        })
      },
      async showDetail(item){
        this.record = item;
        var record = {};
        let {result} = await getAction(this.url.contractList, {contractId: item.relId});
        record = result.list[0];
        if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
          record.filelist = result.list;
        }
        if(record.elecFileRel != null || record.elecFileRel != "" || record.elecFileRel != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.elecFileRel});
          record.elefilelist = result.list;
        }
        record.mark = "message";
        this.$refs.contractInfoModal.edit(record);
        this.$refs.contractInfoModal.title = "编辑";
      },
      modalFormOk(){
        httpAction(this.url.edit,this.record,"put").then((res)=>{
          if(res.success){
            this.loadData();
          }
        })
      },
      showAllMessage(){
        this.$refs.messageShow.show(this.user.id);
        this.$refs.messageShow.title = "全部消息";
      },
      modalShowOk(){
        this.loadData();
      },
      getQueryStatus(){
        let that = this;
        getAction(this.url.list2).then((res) => {
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
    }
  }
</script>

<style lang="scss" scoped>
  .extra-wrapper {
    line-height: 55px;
    padding-right: 24px;

    .extra-item {
      display: inline-block;
      margin-right: 24px;

      a {
        margin-left: 24px;
      }
    }
  }



</style>