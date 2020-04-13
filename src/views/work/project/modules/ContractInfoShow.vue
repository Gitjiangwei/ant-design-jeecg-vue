<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form  layout="inline">
        <a-row :gutter="24">
          <a-col :span="8" >
            <a-form-item label="合同名称" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-input placeholder="请输入合同名称" v-model="queryParam.contractName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8" style="padding-left: 8px;padding-right: 0px;">
            <a-form-item label="合同类型" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <RencheDictSelectTag v-model="queryParam.contractType" placeholder="请选择合同类型" dictCode="CONTRACTTYPE" />
            </a-form-item>
          </a-col>
          <a-col :span="8" style="padding-left: 8px;padding-right: 0px;">
            <a-form-item label="合同状态" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-select placeholder="请选择合同状态" v-model="queryParam.contractStatus">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option value="2">未签订</a-select-option>
                <a-select-option value="1">已签订</a-select-option>
                <a-select-option value="0">已结束</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24" style="padding-top: 12px;">
          <a-col :span="12">
            <a-form-item label="甲方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-input placeholder="请输入甲方合同编号" v-model="queryParam.contractNoA"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 8px;padding-right: 0px;">
            <a-form-item label="乙方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-input placeholder="请输入乙方合同编号" v-model="queryParam.contractNoB"></a-input>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="10" style="padding: 12px;">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
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
        rowKey="contractId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

      </a-table>
    </div>
    <!-- table区域-end -->
  </a-modal>
</template>

<script>
  import { getAction} from '@/api/manage'
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";
  import {filterObj} from '@/utils/util';
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'

  export default {
    name: "contractInfoShow",
    components: {ACol, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        // 查询条件
        queryParam: {},
        //字典数组缓存
        typeDictOptions: [],
        confirmLoading: false,
        // 表头
        columns: [
          {
            title: '合同名称',
            align: "center",
            dataIndex: 'contractName',
          },

          {
            title: '合同类型',
            align: "center",
            dataIndex: 'contractType',
            customRender: (text, record, index) => {
              //字典值替换通用方法
              return filterDictText(this.typeDictOptions, text);
            }
          },
          {
            title: '甲方公司',
            align: "center",
            dataIndex: 'companyNameA'
          },
          {
            title: '甲方合同编号',
            align: "center",
            dataIndex: 'contractNoA'
          },
          {
            title: '乙方公司',
            align: "center",
            dataIndex: 'companyNameB'
          },
          {
            title: '乙方合同编号',
            align: "center",
            dataIndex: 'contractNoB'
          },
          {
            title: '合同状态',
            align: "center",
            dataIndex: 'contractStatus',
            customRender: (text, record, index) => {
              //字典值替换通用方法
              if(text == '2'){
                return "未签订";
              }else if (text == '1'){
                return "已签订";
              }else if (text == '0'){
                return "已结束";
              }
            }
          }
        ],

        //数据集
        dataSource: [],
        // 分页参数
        ipagination: {
          current: 1,
          pageSize: 30,
          pageSizeOptions: ['20', '30', '40'],
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
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        url: {
          list: "/renche/contractInfo/list",
        },
      }
    },
    created () {
      this.loadData();
      //初始化字典配置
      this.initDictConfig();
    },
    methods: {
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      initDictConfig() {
        //初始化字典 - 合同类型
        initDictOptions('CONTRACTTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
      },
      show () {
        this.queryParam = {};
        this.visible = true;
        this.loadData();
      },
      close () {
        this.selectedRowKeys = [];
        this.selectionRows = [];
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条数据！');
          return;
        } else {
          this.selectedRowKeys = [];
          this.selectionRows = [];
          this.$emit('func',this.selectedRows);
          this.close();
        }
      },
      handleCancel () {
        this.close()
      },
      getQueryParams() {
        var param = Object.assign({}, this.queryParam, this.isorter);
        param.field = this.getQueryField();
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
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
        if(selectionRows.length > 1){
          var selectKey = this.selectedRowKeys[1];
          var selectRow0 = this.selectedRows[0];
          var selectRow1 = this.selectedRows[1];
          this.selectedRowKeys = [];
          this.selectedRowKeys.push(selectKey);
          this.selectedRows = [];
          if(selectKey == selectRow0.tenderId){
            this.selectedRows.push(selectRow0);
          }else{
            this.selectedRows.push(selectRow1);
          }

        }
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
        that.loadData(1);
      }

    }
  }
</script>

<style scoped>

</style>