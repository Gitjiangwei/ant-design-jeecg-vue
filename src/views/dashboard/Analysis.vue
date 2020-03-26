<template>
  <div class="page-header-index-wide">
    <a-row :gutter="24">
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="总销售额" total="￥126,560">
          <a-tooltip title="指标说明" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <div>
            <trend flag="up" style="margin-right: 16px;">
              <span slot="term">周同比</span>
              12%
            </trend>
            <trend flag="down">
              <span slot="term">日同比</span>
              11%
            </trend>
          </div>
          <template slot="footer">日均销售额<span>￥ 234.56</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="访问量" :total="8846 | NumberFormat">
          <a-tooltip title="指标说明" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <div>
            <mini-area />
          </div>
          <template slot="footer">日访问量<span> {{ '1234' | NumberFormat }}</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="支付笔数" :total="6560 | NumberFormat">
          <a-tooltip title="指标说明" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <div>
            <mini-bar />
          </div>
          <template slot="footer">转化率 <span>60%</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="运营活动效果" total="78%">
          <a-tooltip title="指标说明" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <div>
            <mini-progress color="rgb(19, 194, 194)" :target="80" :percentage="78" height="8px" />
          </div>
          <template slot="footer">
            <trend flag="down" style="margin-right: 16px;">
              <span slot="term">同周比</span>
              12%
            </trend>
            <trend flag="up">
              <span slot="term">日环比</span>
              80%
            </trend>
          </template>
        </chart-card>
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
  import ChartCard from '@/components/ChartCard'
  import ACol from "ant-design-vue/es/grid/Col"
  import ATooltip from "ant-design-vue/es/tooltip/Tooltip"
  import MiniArea from '@/components/chart/MiniArea'
  import MiniBar from '@/components/chart/MiniBar'
  import MiniProgress from '@/components/chart/MiniProgress'
  import RankList from '@/components/chart/RankList'
  import Bar from '@/components/chart/Bar'
  import Trend from '@/components/Trend'
  import {getLoginfo} from '@/api/api.js'
  import { getAction, httpAction} from '@/api/manage'
  import ContractInfoModal from '../work/project/modules/ContractInfoModal'
  import MessageShow from '../work/message/MessageShow'

  const rankList = []
  for (let i = 0; i < 7; i++) {
    rankList.push({
      name: '白鹭岛 ' + (i+1) + ' 号店',
      total: 1234.56 - i * 100
    })
  }

  export default {
    name: "Analysis",
    components: {
      ATooltip,
      ACol,
      ChartCard,
      MiniArea,
      MiniBar,
      MiniProgress,
      RankList,
      Bar,
      Trend,
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
      }
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

  /* 首页访问量统计 */
  .head-info {
    position: relative;
    text-align: left;
    padding: 0 32px 0 0;
    min-width: 125px;

    &.center {
      text-align: center;
      padding: 0 32px;
    }

    span {
      color: rgba(0, 0, 0, .45);
      display: inline-block;
      font-size: .95rem;
      line-height: 42px;
      margin-bottom: 4px;
    }
    p {
      line-height: 42px;
      margin: 0;
      a {
        font-weight: 600;
        font-size: 1rem;
      }
    }
  }
</style>