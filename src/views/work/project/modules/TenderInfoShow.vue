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
          <a-col :span="8">
            <a-form-item label="项目名称" :wrapperCol="wrapperCol" :labelCol="labelCol"  style="width: 100%;">
              <a-input placeholder="请输入项目名称" v-model="queryParam.prjName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8" style="padding-left: 8px;padding-right: 0px;">
            <a-form-item label="招标编号" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-input placeholder="请输入招标编号" v-model="queryParam.tenderNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8" style="padding-left: 8px;padding-right: 0px;">
            <a-form-item label="投标单位" :wrapperCol="wrapperCol" :labelCol="labelCol" style="width: 100%;">
              <a-input placeholder="请输入投标单位" v-model="queryParam.tenderCompany"></a-input>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col  :span="10" style="padding: 12px;">
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
        rowKey="tenderId"
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

  export default {
    name: "tenderInfoShow",
    components: {ACol, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        // 查询条件
        queryParam: {},
        confirmLoading: false,
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '项目名称',
            align: "center",
            dataIndex: 'prjName',
          },
          {
            title: '招标编号',
            align: "center",
            dataIndex: 'tenderNo',
          },
          {
            title: '投标单位',
            align: "center",
            dataIndex: 'tenderCompany',
          },
          {
            title: '报价（万元）',
            align: "center",
            dataIndex: 'tenderOffer',
          },
          {
            title: '保证金（万元）',
            align: "center",
            dataIndex: 'deposit',

          },
          {
            title: '保证金是否退回',
            align: "center",
            dataIndex: 'isBack',
          },
          {
            title: '创建时间',
            align: "center",
            dataIndex: 'createTime',
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
          list: "/renche/tender/qrytenderList",
        },
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
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      show () {
        this.visible = true;
        this.queryParam = {};
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