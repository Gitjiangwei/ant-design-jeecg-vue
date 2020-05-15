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
            <a-form-item label="客户名称">
              <a-input placeholder="请输入客户名称" maxlength="15" v-model="queryParam.companyName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="客户类型">
              <RencheDictSelectTag v-model="queryParam.type" placeholder="请选择客户类型" dictCode="CUSTOMETYPE"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>

              </span>
          </a-col>
        </a-row>
      </a-form>
    </div>


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
        rowKey="companyId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio'}"
        @change="handleTableChange">


      </a-table>

    </div>
  </a-modal>


</template>

<script>
    //import CompanyInfoModal from '././modules/CompanyInfoModal'
    import {filterObj} from '@/utils/util'
    import {deleteAction, getAction, postAction} from '@/api/manage'
    import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
   // import FileDetail from "./modules/FileDetail";

    export default {
      name: "companyInfoShow",
      components: {
      /*  FileDetail,
        CompanyInfoModal,*/
      },
      data() {
        return{
          description: '客户信息管理页面',

          // 查询条件
          queryParam: {},
          //字典数组缓存
          typeDictOptions: [],
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
              title: '客户名称',
              align: "center",
              dataIndex: 'companyName'
            },
            {
              title: '客户类型',
              align: "center",
              dataIndex: 'type',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                return filterDictText(this.typeDictOptions, text);
              }
            },
            {
              title: '税号',
              align: "center",
              dataIndex: 'shuihao'
            },
            {
              title: '所在地址',
              align: "center",
              dataIndex: 'address'
            },
            {
              title: '联系人',
              align: "center",
              dataIndex: 'contacts'
            },
            {
              title: '联系电话',
              align: "center",
              dataIndex: 'phone'
            },
            {
              title: '附件',
              align: "center",
              dataIndex: 'fileCount',
            },

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
          visible:false,
          confirmLoading: false,
          url: {
            list: "/renche/companyInfo/list",
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
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
        },
        handleOk () {
          if (this.selectedRowKeys.length <= 0) {
            this.$message.warning('请选择一条数据！');
            return;
          } else {
          /*  this.$emit('func',this.selectedRows);
            this.selectedRowKeys = [];
            this.selectionRows = [];
            this.close();*/

            this.$emit('func',this.selectedRows[0]);
            this.close();
          }
        },

        show(){
          this.visible = true;
          this.queryParam = {};
          this.loadData(1);
        },
        initDictConfig() {
          //初始化字典 - 客戶類型
          initDictOptions('CUSTOMETYPE').then((res) => {
            if (res.success) {
              this.typeDictOptions = res.result;
            }
          });
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

        searchReset() {
          var that = this;
          that.queryParam = {}
          that.loadData(1);
        },
        onSelectChange(selectedRowKeys, selectionRows) {
          this.selectedRowKeys = selectedRowKeys;
          this.selectedRows = selectionRows;
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
              ids += this.selectedRowKeys[a] + ",";
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
        handleCancel() {
          this.close()
        },
        close () {
          this.selectedRowKeys = [];
          this.selectionRows = [];
          this.$emit('close');
          this.visible = false;
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