<template>
  <a-card :bordered="false" >
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="6">
            <a-form-item label="设备编号">
              <a-input placeholder="请输入设备编号" maxlength="15" v-model="queryParam.equipNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="设备状态">
             <!-- <RencheDictSelectTag v-model="queryParam.equipStatus" placeholder="请选择设备状态" dictCode="ELECTSTATUS"/>-->
                <a-select v-model="queryParam.equipStatus">
                  <a-select-option :key="1">空闲</a-select-option>
                  <a-select-option :key="2">维修中</a-select-option>
                </a-select>
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

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;margin-top: 15px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="equipId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleRepair(record)">维修</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定将设备报废吗?" @confirm="() => handleBaoF(record)">
                  <a>报废</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定设备可以投入使用吗?" @confirm="() => handleStatus(record)">
                  <a>维修完成</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
      <purchase-info-stock-edit ref="purchaseInfoStockEdit" @ok="modalFormOk"></purchase-info-stock-edit>
    </div>
  </a-card>

</template>

<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import {deleteAction, getAction, httpAction} from '@/api/manage';
  import {filterObj} from '@/utils/util';
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil';
  import purchaseInfoStockEdit  from "./modules/PurchaseInfoStockEdit.vue";

  export default {
    name: "PurchaseStackDetail",
    components: {
      ARow,
      purchaseInfoStockEdit
    },
    data() {
      return{
        description: '采购管理页面',

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
            width: 40,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '库存设备名称',
            align: "center",
            dataIndex: 'materialName'
          },
          {
            title: '库存设备编号',
            align: "center",
            dataIndex: 'equipNo'
          },
          {
            title: '厂家设备编号',
            align: "center",
            dataIndex: 'manufacoryNo'
          },
          {
            title:"拥有方式",
            align:"center",
            dataIndex: "haveWay",
            width: 80,
            customRender: (text) => {
              if(text == '0'){
                return "租赁";
              }else if(text == '1'){
                return "购买";
              }else {
                return text;
              }
            }
          },
          {
            title:"当前状态",
            align:"center",
            dataIndex: "equipStatus",
            width: 80,
            customRender: (text, record, index) => {
              //字典值替换通用方法
              return filterDictText(this.statDictOptions, text);
            }
          },
          {
            title:"使用次数",
            align:"center",
            dataIndex: "useTimes",
            width: 80,
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            width: 120,
            scopedSlots: {customRender: 'action'},
          }
        ],
        materialId:"",
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
          list: "/renche/equip/equipKey",
          equipStatus: "/renche/equip/updateStatus",
          repair: "/renche/equip/updateStatusweix",
          baof:"/renche/equip/updateEquipbaof",
          delete: "/renche/equip/delete",
        },
      }
    },
    created() {
      this.materialId = this.$route.params.materialId;
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
        params.materialId = this.materialId;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },

      initDictConfig: function() {
        //初始化字典 - 设备状态
        initDictOptions('ELECTSTATUS').then((res) => {
          if (res.success) {
            this.statDictOptions = res.result;
            console.log(this.statDictOptions);
          }
        });
      },
      handleEdit: function (record) {
        this.$refs.purchaseInfoStockEdit.edit(record);
        this.$refs.purchaseInfoStockEdit.title = "编辑";
      },
      handleRepair: function(record){
        var that = this;
        if(record.equipStatus != "FREE"){
          that.$message.warning("只能维修空闲中的设备！")
          return;
        }
        let httpurl = this.url.repair;
        deleteAction(httpurl,{equipId: record.equipId}).then((res)=>{
            if(res.success){
              that.$message.success(res.message);
              that.loadData();
            }else{
              that.$message.warning(res.message);
              that.loadData();
            }
         }).finally(() => {
            that.confirmLoading = false;
            that.close();
        })
      },
      handleBaoF: function(record){
        var that = this;
        let httpurl = this.url.baof;
        if (parseInt(record.useTimes)< 3) {
          this.$confirm({
            title: "确认报废",
            content: "该设备还没有达到最大使用次数，确定要报废吗？",
            onOk: function () {
              deleteAction(httpurl,{equipId: record.equipId}).then((res)=>{
                if(res.success){
                  that.$message.success(res.message);
                  that.loadData();
                }else{
                  that.$message.warning(res.message);
                  that.loadData();
                }
              }).finally(() => {
                that.confirmLoading = false;
                that.close();
              })
            }
          });
        }else {
          deleteAction(httpurl,{equipId: record.equipId}).then((res)=>{
            if(res.success){
              that.$message.success(res.message);
              that.loadData();
            }else{
              that.$message.warning(res.message);
              that.loadData();
            }
          }).finally(() => {
            that.confirmLoading = false;
            that.close();
          })
        }

      },
      handleStatus: function(record){
        var that = this;
        if(record.equipStatus != "INREPAIR"){
          that.$message.warning("只能将维修中的设备投入使用！")
          return;
        }
        deleteAction(that.url.equipStatus, {equipId: record.equipId}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },
      modalFormOk() {
        // 新增/修改 成功时，重载列表
        this.loadData();
      },
      searchReset() {
        var that = this;
        that.queryParam = {}
        that.loadData(1);
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
    }
  }

</script>

<style scoped>

</style>