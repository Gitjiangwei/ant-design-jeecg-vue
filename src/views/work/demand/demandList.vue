<template>
  <a-card :bordered="false" >
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

      <!-- 操作按钮区域 -->
      <div class="table-operator">
        <a-button @click="handleAdd" type="primary" icon="plus">新增 </a-button>
        <!--   <a-button type="primary" style="margin-left: 10px" >
             <a :href="'http://localhost:8080/jeecg-boot/renche/purchase/exportPurchase'" target="_blank" style="margin-left: 10px">导出</a>
           </a-button>-->
        &nbsp;
        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1" @click="batchDel()">
              <a-icon type="delete"/>
              删除
            </a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px"> 批量操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>
      </div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;margin-top: 15px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

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
            <a @click="updateStatus(record)">编辑</a>
                    <a-divider type="vertical"/>
                <a-dropdown>
                  <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
                  <a-menu slot="overlay">
                    <a-menu-item>
                      <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record)">
                        <a>删除</a>
                      </a-popconfirm>
                    </a-menu-item>
                    <a-menu-item>
                      <a-popconfirm title="确定发送吗?" @confirm="() => handleSendOut(record)">
                        <a>发送</a>
                      </a-popconfirm>
                    </a-menu-item>
                    <a-menu-item v-if="record.isSend=='3'">
                        <a @click="handleReasons(record)">退回原因</a>
                    </a-menu-item>
                  </a-menu>
                </a-dropdown>
        </span>
      </a-table>
    </div>
    <demand-modules ref="DemandModules" @ok="modalFormOk"></demand-modules>
    <demand-resons-modules ref="DemandResonsModules"></demand-resons-modules>
  </a-card>
</template>
<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import {deleteAction, getAction, httpAction} from '@/api/manage';
  import DemandModules from "./modules/demandModules";
  import DemandResonsModules from "./modules/demandResonsModules"
  import {filterObj} from '@/utils/util';

  export default {
    name: "EquipInfoDetail",
    components: {
      ARow,
      DemandModules,
      DemandResonsModules
    },
    data() {
      return{
        description: '设备需求管理',
        title: "操作",
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
              }else if (text==0) {
                return "未发送"
              }else {
                return text;
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
          list: "/renche/demand/queryDemand",
          delete:"/renche/deamand/delDemand",
          sendOut:"renche/demand/updateStatus",
          deletes:"renche/demand/delDemands",
        },
      }
    },
    created() {
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
      batchDel: function () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        }else {
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            if(this.selectionRows[a].isSend == "1" || this.selectionRows[a].isSend == "2"){
              this.$message.warning('您要删除的设备需求单中包含未处理或已经处理的需求单，只能删除未发送和退回的设备需求！');
              return;
            }
            ids += this.selectionRows[a].demandId + ",";
          }
          var that = this;
          var title = "确认删除";
          var content = "是否删除选中数据";
          var url = that.url.deletes;
          this.$confirm({
            title: title,
            content: content,
            onOk: function () {
              deleteAction(url, {demandIds: ids}).then((res) => {
                if (res.success) {
                  that.$message.success(res.message);
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
      handleAdd:function(){
        this.$refs.DemandModules.add();
        this.$refs.DemandModules.title="新增设备需求";
      },
      updateStatus:function(record){
        if(record.isSend=="1"|| record.isSend == "2"){
          this.$message.warning("只能编辑未发送和退回的设备需求！");
          return;
        }
        this.$refs.DemandModules.edit(record);
        this.$refs.DemandModules.title="编辑设备需求";
      },
      handleReasons:function(record){
        this.$refs.DemandResonsModules.edit(record);
        this.$refs.DemandResonsModules.title="查看退回原因";
      },
      handleSendOut:function(record){
        if(record.isSend == "1"){
          this.$message.warning("该设备需已经发送过，无法再次发送");
          return;
        }
        if(record.isSend == "2"){
          this.$message.warning("该设备需已经处理完成，无法再次发送");
          return;
        }
        var that = this;
        deleteAction(that.url.sendOut,{demandId:record.demandId,status:"1"}).then((res)=>{
          if(res.success){
            that.$message.success(res.message);
            that.loadData();
            that.onClearSelected();
          }else {
            that.$message.warning(res.message);
          }
        })
      },
      handleDelete:function(record){
        if(record.isSend=="1"||record.isSend=="2"){
          this.$message.warning("只能删除未发送和退回的设备需求！");
          return;
        }
        var that = this;
        deleteAction(this.url.delete,{demandId:demandId}).then((res) =>{
          if(res.success){
            that.$message.success(res.message);
            that.loadData();
            that.onClearSelected();
          }else {
            that.$message.warning(res.message);
          }
        })
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