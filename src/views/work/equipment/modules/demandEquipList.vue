<template>
  <a-modal
    :title="title"
    :width="1000"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
    @ok="handleCancel"
    cancelText="关闭"
  >
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="12">
            <a-form-item label="设备名称">
              <a-input placeholder="请输入设备名称" maxlength="30" v-model="queryParam.equipmentName"></a-input>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <!--<a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>-->
              </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <div>
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <!-- 操作按钮区域 -->
        <span slot="action" slot-scope="text, record">
            <a @click="updateStatus(record)">处理</a>
            <a-divider type="vertical"/>
            <a @click="handleReasons(record)">退回</a>
        </span>
      </a-table>
      <demand-resons-modules ref="DemandResonsModules" @ok="modalFormOk"></demand-resons-modules>
    </div>
  </a-modal>
</template>
<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import {deleteAction, getAction, httpAction} from '@/api/manage';
  import DemandResonsModules from "../../demand/modules/demandResonsModules";
  import {filterObj} from '@/utils/util';

  export default {
    name: "DemandList",
    components: {
      DemandResonsModules,
      ARow,
    },
    data() {
      return{
        description: '采购管理页面',
        timer:"",
        purchaseId:"",
        title: "操作",
        visible: false,
        confirmLoading: false,
        // 查询条件
        queryParam: {},
        //字典数组缓存
        statDictOptions: [],
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
            title: '设备名称',
            align: "center",
            dataIndex: 'equipmentName'
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipmentModel'
          },
          {
            title: '需要采购的设备数量',
            align: "center",
            dataIndex: 'equipmentNumber'
          },
          {
            title: '提交人',
            align: "center",
            dataIndex: 'createName'
          },
          {
            title: '处理时间',
            align: "center",
            dataIndex: 'whetherTime'
          },
          {
            title:"状态",
            align:"center",
            dataIndex: "isSend",
            customRender:function (text) {
              if(text==1){
                return "未处理";
              }else if(text==2){
                return "已处理";
              }else if(text==3){
                return "退回";
              }else {
                return text
              }
            }
          },
          {
            title:"备注",
            align:"center",
            dataIndex: "remarks",
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        purchaseId:"",
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
          list: "/renche/demand/queryDemandStatus",
          updateStatus:"/renche/demand/updateStatus",
        },
      }
    },
    methods:{
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        debugger;
        var params = this.getQueryParams();//查询条件
        params.purchaseId = this.purchaseId;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      load:function(record){
        debugger;
        this.visible = true;
        this.loadData(1);
      },
      updateStatus:function(record){
        var that = this;
        var demandId = record.demandId;
        if (record.isSend=='2'){
          that.$message.warning("该设备需求单已处理完成，不需要重复处理！");
          return
        }
        if (record.isSend=='3'){
          that.$message.warning("该设备需求单已退回，不能进行处理！");
          return
        }
        deleteAction(that.url.updateStatus, {demandId: demandId,status:'2'}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.loadData();
            that.onClearSelected();
          } else {
            that.$message.warning(res.message);
          }
        });
      },
      handleReasons:function(record){
        if(record.isSend!="1"){
          this.$message.warning("只能退回未处理的设备需求单！");
          return
        }
        this.$refs.DemandResonsModules.add(record.demandId);
        this.$refs.DemandResonsModules.title="退回原因";
      },
      handleCancel() {
        this.close();
      },
      close() {
        this.$emit('ok');
        this.visible = false;
      },
      modalFormOk() {
        // 新增/修改 成功时，重载列表
        this.loadData();
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
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectionRows = selectionRows;
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
      },
      searchQuery(){
        this.loadData(1);
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
      handleCancel() {
        this.close();
      },
      close() {
        this.$emit('ok');
        this.visible = false;
      },
    }
  }
</script>