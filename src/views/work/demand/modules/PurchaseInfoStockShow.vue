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
      <!--    <a-col :span="6">
            <a-form-item label="设备名称" >
              <a-input placeholder="请输入设备名称" maxlength="30" v-model="queryParam.equipName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="设备型号">
              <a-input placeholder="请输入设备型号" maxlength="15" v-model="queryParam.equipModel"></a-input>
            </a-form-item>
          </a-col>-->
        </a-row>
        <a-row>
          <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                &nbsp;
               <!-- <a-button type="primary" @click="addDemand" icon="plus" >新增设备需求</a-button>-->
              </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- table区域-begin -->
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
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleDetail(record)">详情</a>

        </span>

      </a-table>
    </div>

    <purchase-detail-show ref="PurchaseDetailShow"  @ok="modalFormOk"></purchase-detail-show>
    <demand-modules ref="DemandModules"></demand-modules>
  </a-modal>





</template>

<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";
  import PurchaseDetailShow from "./PurchaseDetailShow";
  import DemandModules from "../../demand/modules/demandModules";
  import {deleteAction, getAction, postAction} from '@/api/manage';
  import {filterObj} from '@/utils/util';


  export default {
    name: "PurchaseInfoStockShow",
    components: {ACol, ARow,PurchaseDetailShow,DemandModules},
    data() {
      return{
        description: '设备库存页面',
        visible: false,
        title: "操作",
        projectId:"",
        // 查询条件
        queryParam: {},
        confirmLoading: false,
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
            title: '设备名称',
            align: "center",
            dataIndex: 'equipName',
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipModel'
          },
          {
            title: '数量',
            align: "center",
            dataIndex: 'count'
          },
          {
            title: '使用中',
            align: "center",
            dataIndex: 'inuseCount',

          },
          {
            title:"空闲",
            align:"center",
            dataIndex: "freeCount",
          },
          {
            title:"报废",
            align:"center",
            dataIndex: "scripCount",
          },
          {
            title:"维修中",
            align:"center",
            dataIndex: "maintenonceCount",
          },
          {
            title:"入库时间",
            align:"center",
            dataIndex: "createTime",
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
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
        equipmentName:"",
        equipmentModel:"",
        url: {
           list: "/renche/equip/equipList",
        },
      }
    },
    created() {
      this.loadData();
    },
    methods: {
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        console.log(params);
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      show (record,equipmentName,equipmentModel) {
        this.equipmentName=equipmentName;
        this.equipmentModel=equipmentModel;
        this.projectId = record;
        this.visible = true;
        this.loadData();
      },

      close () {
        this.$emit('close');
        this.visible = false;
      },

      handleOk () {
        this.close();
      },
      handleCancel() {
        this.close()
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleDetail: function(record){
        //this.$router.push({ name: 'work-equipment-PurchaseStackDetail',params:{purchaseId:record.purchaseId} })
        this.$refs.PurchaseDetailShow.show(record,this.projectId,this.equipmentName,this.equipmentModel);
        this.$refs.PurchaseDetailShow.title = "设备详情";

      },

/*      handleDelete: function (id) {
        var that = this;
        deleteAction(that.url.delete, {id: id}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },*/
      modalFormOk() {
        // 新增/修改 成功时，重载列表
        //this.loadData();
        this.$emit('ok');
        this.close();
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
        param.equipName=this.equipmentName;
        param.equipModel=this.equipmentModel;
        console.log(param)
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