<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
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
              <RencheDictSelectTag v-model="queryParam.equipStatus" placeholder="请选择设备状态" dictCode="ELECTSTATUS"/>
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
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

      </a-table>
    </div>
  </a-modal>

</template>

<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import {deleteAction, getAction, httpAction} from '@/api/manage';
  import {filterObj} from '@/utils/util';
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil';
  export default {
    name: "PurchaseDetailShow",
    components: {
      ARow
    },
    data() {
      return{
        title: "操作",
        visible:false,
        confirmLoading: false,
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
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '库存设备名称',
            align: "center",
            dataIndex: 'equipName'
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
            title: '设备情况',
            align: "center",
            dataIndex: 'condition'
          },
          {
            title:"当前状态",
            align:"center",
            dataIndex: "equipStatus",
            customRender: (text, record, index) => {
              //字典值替换通用方法
              return filterDictText(this.statDictOptions, text);
            }
          },
          {
            title:"使用次数",
            align:"center",
            dataIndex: "useTimes",
          }
        ],
        purchaseId:"",
        projectId:"",
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
        equipmentName:"",
        equipmentModel:"",
        url: {
          list: "/renche/equip/equipKeyDetail",
          outEquip: "/renche/outEquip/inserOutEquip",
          repair: "/renche/equip/updateDetailEquipInfo"
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
        var params = this.getQueryParams();//查询条件
        params.purchaseId = this.purchaseId;
        params.status=
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      show:function(record,projectId,equipmentName,equipmentModel){
        this.equipmentName=equipmentName;
        this.equipmentModel=equipmentModel;
        this.visible = true;
        this.projectId = projectId;
        this.purchaseId = record.purchaseId;
        this.loadData(1);
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
      handleRepair: function(record){
        var that = this;
        debugger;
        let httpurl = this.url.repair;
        let method = 'post';
        httpAction(httpurl,record,method).then((res)=>{
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

      handleOk(){
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条数据！');
          return;
        } else {
          var that = this;
          that.confirmLoading = true;
          let httpurl = this.url.outEquip;
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            if(this.selectionRows[a].equipStatus == "FREE") {
              ids += this.selectionRows[a].equipId + ",";
            }else{
              this.$message.warning('只能选择空闲中的设备！');
              return;
            }
            if (this.selectionRows[a].usrTimes > "3"){
              this.$message.warning('您选择的设备有使用过3次的设备');
            }
          }

          this.$confirm({
            title: "关联设备",
            content: "提交后无法修改，是否确定关联？",
            onOk: function () {
              deleteAction(httpurl,{equipIds: ids, prjItemId: that.projectId}).then((res)=> {
                if (res.success) {
                  that.$message.success(res.message);
                  that.$emit('ok');
                } else {
                  that.$message.warning(res.message);
                }
              }).finally(() => {
                that.confirmLoading = false;
                that.selectedRowKeys = [];
                that.selectionRows = [];
                that.close();
              });
            }
          });












        }
      },
      handleCancel() {
        this.close()
      },
      close() {
        this.$emit('close');
        this.visible = false;
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