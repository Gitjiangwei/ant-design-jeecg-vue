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
            <a-form-item label="设备编号">
              <a-input placeholder="请输入设备编号" maxlength="15" v-model="queryParam.equipNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
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

    <!-- 操作按钮区域 -->
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
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <!--          <span slot="purchaseItem" slot-scope="text,record">-->
        <!--              <a @click="handleEdit(record)">chaolianj(record.purchaseItem)</a>-->
        <!--          </span>-->


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
    name: "EquipInfoDetail",
    components: {
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
          list: "/renche/equip/equipKeyDetail",
        },
      }
    },
    created() {
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
      detail:function(record){
        this.visible = true;
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