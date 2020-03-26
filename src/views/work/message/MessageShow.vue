<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleCancel"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form  layout="inline">
        <a-row :gutter="24">
          <a-col :span="12" style="padding-left: 12px;padding-right: 0px;">
            <a-form-item label="消息时间" >
              <a-date-picker v-model="queryParam.startDate" placeholder="Start"/>
              --<a-date-picker v-model="queryParam.endDate" placeholder="End"/>
            </a-form-item>
          </a-col>
          <a-col :span="8" style="padding-left: 12px;padding-right: 0px;">
            <a-form-item label="消息状态" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
                <a-select v-model="queryParam.messageStatus" placeholder="请选择消息状态">
                  <a-select-option value="">请选择</a-select-option>
                  <a-select-option value="0">已读</a-select-option>
                  <a-select-option value="1">未读</a-select-option>
                </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="10" style="padding: 12px;">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="relcheckoad" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="allRead" icon="check" style="margin-left: 8px">一键已读</a-button>
              <a-dropdown v-if="selectedRowKeys.length > 0">
                <a-menu slot="overlay">
                  <a-menu-item key="1" @click="batchDel">
                    <a-icon type="delete"/>
                    删除
                  </a-menu-item>
                </a-menu>
                <a-button style="margin-left: 8px"> 批量操作
                  <a-icon type="down"/>
                </a-button>
              </a-dropdown>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>

      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="messageId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

          <a slot="messageContent" slot-scope="text, record" class="a_style" @click="showDetail(record)" >{{text}}</a>

         <span slot="action" slot-scope="text, record">
           <a @click="handleDel(record.messageId)">删除</a>
         </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <contract-info-modal ref="contractInfoModal" @ok="modalFormOk"></contract-info-modal>

  </a-modal>
</template>

<script>
  import { getAction,httpAction,deleteAction} from '@/api/manage'
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";
  import {filterObj} from '@/utils/util';
  import ARangePicker from "ant-design-vue/es/date-picker/RangePicker";
  import moment from "moment"
  import ContractInfoModal from '../project/modules/ContractInfoModal'

  export default {
    name: "messageShow",
    components: {ARangePicker, ACol, ARow,ContractInfoModal},
    data () {
      return {
        title:"操作",
        visible: false,
        // 查询条件
        queryParam: {},
        reciveUser: "",
        confirmLoading: false,
        record: {},
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '消息内容',
            align: "center",
            dataIndex: 'messageContent',
            scopedSlots: { customRender: 'messageContent' }
          },
          {
            title: '消息类型',
            align: "center",
            dataIndex: 'messageType',
            customRender: (text, record, index) => {
              if(text == '1'){
                return '合同催款提醒';
              }
            }
          },
          {
            title: '消息状态',
            align: "center",
            dataIndex: 'messageStatus',
            customRender: (text, record, index) => {
              if(text == '0'){
                return '已读';
              }else{
                return '未读';
              }
            }
          },
          {
            title: '创建日期',
            align: "center",
            dataIndex: 'createTime'
          },
          {
            title: '操作',
            align: "center",
            dataIndex: 'action',
            scopedSlots: {customRender: 'action'}
          }
        ],

        //数据集
        dataSource: [],
        // 分页参数
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        isorter: {
          column: 'createTime',
          order: 'desc',
        },
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: "/renche/messageInfo/list",
          edit: "/renche/messageInfo/edit",
          delete: "/renche/messageInfo/delete",
          deleteBatch: "/renche/messageInfo/deleteBatch",
          batchEditRead:"/renche/messageInfo/batchEditRead",
          contractList: "/renche/contractInfo/list",
          filelist: "/renche/purchase/fileList",
        },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        }
      }
    },
    created () {
      this.loadData();
    },
    methods: {
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        if(params.startDate != undefined){
          var startDate = moment(params.startDate).format('YYYY-MM-DD');
          params.startDate = startDate;
        }
        if(params.endDate != undefined){
          var endDate = moment(params.endDate).format('YYYY-MM-DD');
          params.endDate = endDate;
        }
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      show (recode) {
        this.visible = true;
        this.reciveUser = recode;
        this.loadData();
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      getQueryParams() {
        var param = Object.assign({}, this.queryParam, this.isorter);
        param.field = this.getQueryField();
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        param.reciveUser = this.reciveUser;
        return filterObj(param);
      },
      getQueryField() {
        //TODO 字段权限控制
        var str = "id,";
        this.columns.forEach(function (value, index) {
          str += "," + value.dataIndex;
        });
        return str;
      },
      searchQuery() {
        this.loadData(1);
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectedRows = selectionRows;
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
      },
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadData();
      },
      searchReset() {
        var that = this;
        that.queryParam = {}
        that.queryParam.reciveUser = this.reciveUser;
        that.loadData(1);
      },
      batchDel: function () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        } else {
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ",";
          }
          var that = this;
          this.$confirm({
            title: "确认删除",
            content: "是否删除选中数据?",
            onOk: function () {
              deleteAction(that.url.deleteBatch, {ids: ids}).then((res) => {
                if (res.success) {
                  that.$message.success(res.message);
                  that.ipagination.current = 1;
                  that.loadData();
                  that.onClearSelected();
                } else {
                  that.$message.warning(res.message);
                }
              });
            }
          });
        }
      },
      handleDel: function (id) {
        var that = this;
        deleteAction(that.url.delete, {id: id}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.ipagination.current = 1;
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },
      allRead(){
        deleteAction(this.url.batchEditRead,{reciveUser: this.reciveUser},"post").then((res)=>{
          if(res.success){
            this.loadData();
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
        if(this.record.messageStatus == '1'){
          httpAction(this.url.edit,this.record,"put").then((res)=>{
            if(res.success){
              this.loadData();
            }
          })
        }
      },
    }
  }
</script>

<style lang="scss">
  .a_style{
    color: black;
    &:hover {
     color: #00A0E9;
    }
  }
</style>