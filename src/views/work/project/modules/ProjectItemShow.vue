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
          <a-col :span="6" style="width: 27%;padding-left: 8px;padding-right: 0px;">
            <a-form-item label="工程名称">
              <a-input placeholder="请输入工程名称" v-model="queryParam.prjItemName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6" style="width: 27%;padding-left: 8px;padding-right: 0px;">
            <a-form-item label="项目名称">
              <a-input placeholder="请输入项目名称" v-model="queryParam.prjName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6" style="width: 18%;padding-left: 8px;padding-right: 0px;">
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
        rowKey="prjItemId"
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
  import { getAction,httpAction} from '@/api/manage'
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";
  import {filterObj} from '@/utils/util';
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'

  export default {
    name: "projectItemShow",
    components: {ACol, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        // 查询条件
        queryParam: {},
        contractId: "",
        confirmLoading: false,
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
            title: '工程编号',
            align: "center",
            dataIndex: 'prjItemNum'
          },
          {
            title: '工程名称',
            align: "center",
            dataIndex: 'prjItemName'
          },
          {
            title: '工程类型',
            align: "center",
            dataIndex: 'prjItemType',
            customRender: (text, record, index) => {
              //字典值替换通用方法
              return filterDictText(this.typeDictOptions, text);
            }
          },
          {
            title: '项目名称',
            align: "center",
            dataIndex: 'prjName'
          },
          {
            title: '所属公司',
            align: "center",
            dataIndex: 'companyName'
          },
          {
            title: '负责人',
            align: "center",
            dataIndex: 'personInCharge'
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
          list: "/renche/contractInfo/allPrjItemWithoutContractList",
          add: "/renche/contractInfo/addProjectItem"
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
        if(this.contractId != ""){
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
        }
      },
      initDictConfig() {
        //初始化字典 - 工程类型
        initDictOptions('PROJECTITEMTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
      },
      show (recode) {
        this.visible = true;
        this.contractId = recode;
        this.loadData();
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条数据！');
          return;
        } else {
          var that = this;
          that.confirmLoading = true;
          let httpurl = this.url.add;
          let method = 'post';
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ",";
          }

          httpAction(httpurl,{contractId: this.contractId, prjItemIds: ids},method).then((res)=>{
            if(res.success){
              that.$message.success(res.message);
              this.$emit('func');
            }else{
              that.$message.warning(res.message);
            }
          }).finally(() => {
            that.confirmLoading = false;
            that.selectedRowKeys = [];
            that.selectionRows = [];
            that.close();
          })
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
        param.contractId = this.contractId;
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
        that.loadData(1);
      }

    }
  }
</script>

<style scoped>

</style>