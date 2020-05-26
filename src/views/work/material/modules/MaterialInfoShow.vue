<template>
  <a-card :bordered="false">
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="6">
            <a-form-item label="物料编号">
              <a-input placeholder="请输入物料编号" v-model="queryParam.materialNo"  maxlength="30"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="物料名称">
              <a-input placeholder="请输入物料名称" v-model="queryParam.materialName"  maxlength="30"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="物料型号">
              <a-input placeholder="请输入物料型号" v-model="queryParam.materialType"  maxlength="30"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
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
        rowKey="materialId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

          <span slot="action" slot-scope="text, record">
             <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.materialId )">
                <a>删除</a>
              </a-popconfirm>
          </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->

    <MaterialInfoModal ref="materialInfoModal" @ok="modalFormOk"></MaterialInfoModal>

  </a-card>

</template>

<script>
  import ARow from "ant-design-vue/es/grid/Row";
  import MaterialInfoModal from './MaterialInfoModal';
  import {filterObj} from '@/utils/util';
  import {deleteAction, getAction, postAction } from '@/api/manage';
  import Vue from 'vue';
  import {ACCESS_TOKEN} from "@/store/mutation-types";

  export default {
    name: "MaterialInfo",
    components: {
      MaterialInfoModal,
      ARow,
    },
    data() {
      return {
        description: '物料库管理',
        // 查询条件
        queryParam: {},
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
            title: '物料编号',
            align: "center",
            dataIndex: 'materialNo',
            width: 200,
          },
          {
            title: '物料名称',
            align: "center",
            dataIndex: 'materialName',
            width: 200,
          },
          {
            title: '拼音',
            align: "center",
            dataIndex: 'pyName',
            width: 150,
          },
          {
            title: '物料型号',
            align: "center",
            dataIndex: 'materialType',
            width: 200,
          },
          {
            title: '物料单位',
            align: "center",
            dataIndex: 'materialUnit',
            width: 100,
          }
        ],
        headers: {},
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
         // visitor:'',
        },
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: "/renche/materialInfo/list",
          delete: "/renche/materialInfo/delete",
          deleteBatch: "/renche/materialInfo/deleteBatch",
          export: "/renche/WorkSerivice/exportWorkService",
          filelist: "/renche/purchase/fileList",

        },
      }
    },
    created() {
      this.loadData();
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
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
            console.log(this.dataSource);
            this.ipagination.total = res.result.total;

          }
        })
      },
      getQueryParams() {
        var param = Object.assign({}, this.queryParam, this.isorter );
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
      superQuery() {
        this.$refs.superQueryModal.show();
      },
      searchReset() {
        var that = this;
        that.queryParam = {}
        that.loadData(1);
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectionRows = selectionRows;
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
      },
      batchDel: function () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        } else {
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {

            ids += this.selectionRows[a].visitId + ",";
          }
          var that = this;
          this.$confirm({
            title: "确认删除",
            content: "是否删除选中数据?",
            onOk: function () {
              deleteAction(that.url.deleteBatch, {ids: ids}).then((res) => {
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
      handleAdd: function () {
        this.$refs.materialInfoModal.add();
        this.$refs.materialInfoModal.title = "新增";
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

      modalFormOk() {
        // 新增/修改 成功时，重载列表
        this.loadData();
      },
    }
  }

</script>

<style scoped>
  .ant-card-body .table-operator {
    margin-bottom: 18px;
  }

  .ant-layout-content {
    margin: 12px 16px 0 !important;
  }

  .ant-table-tbody .ant-table-row td {
    padding-top: 15px;
    padding-bottom: 15px;
  }

  .anty-row-operator button {
    margin: 0 5px
  }

  .ant-btn-danger {
    background-color: #ffffff
  }

  .ant-modal-cust-warp {
    height: 100%
  }

  .ant-modal-cust-warp .ant-modal-body {
    height: calc(100% - 110px) !important;
    overflow-y: auto
  }

  .ant-modal-cust-warp .ant-modal-content {
    height: 90% !important;
    overflow-y: hidden
  }

  /** Button按钮间距 */
  .ant-btn {
    margin-left: 3px
  }
</style>