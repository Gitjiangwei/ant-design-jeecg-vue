<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="发票内容" >
                <a-input placeholder="请输入发票内容" v-model="queryParam.content" maxLength="100"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="税号">
                <a-input placeholder="请输入税号" v-model="queryParam.shuihao" maxLength="20"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="合同名称">
                <a-input placeholder="请输入合同名称" v-model="queryParam.contractName" maxLength="50"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="公司名称">
                <a-input placeholder="请输入公司名称" v-model="queryParam.companyName" maxLength="50"></a-input>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24" v-show="isShow">
            <a-col :span="6">
              <a-form-item label="开票时间">
                <a-date-picker  v-model="queryParam.invociTime" />
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24">
            <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <!-- 操作按钮区域 -->
      <div class="table-operator">
        <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>

        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1" @click="batchDel">
              <a-icon type="delete"/>
              删除
            </a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px"> 批量操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>

        <a-button @click="exportDate" type="primary" icon="export">导出</a-button>
      </div>

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
          rowKey="invociId"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="ipagination"
          :loading="loading"
          :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
          @change="handleTableChange">


          <span slot="action" slot-scope="text, record">
             <a @click="fileDeteil(record)">附件</a>
            <a-divider type="vertical"/>

            <a @click="handleEdit(record)">编辑</a>
            <a-divider type="vertical"/>
            <a @click="handleDelete(record.invociId )">删除</a>
          </span>
        </a-table>
      </div>
      <!-- table区域-end -->

      <!-- 表单区域 -->
        <add-invoci-info ref="addInvociInfo" @ok="modalFormOk"></add-invoci-info>
      <invoci-info-file-detail ref="invociInfoFileDetail" @ok="modalFormOk"></invoci-info-file-detail>


    </a-card>

</template>

<script>
    import ARow from "ant-design-vue/es/grid/Row";
    import AddInvociInfo from './modules/AddInvociInfo';
    import {filterObj} from '@/utils/util';
    import InvociInfoFileDetail from "./modules/InvociInfoFileDetail";
    import {deleteAction, getAction, postAction} from '@/api/manage';

    export default {
      name: "invoicInfo",
      components: {
        ARow,
        AddInvociInfo,
        InvociInfoFileDetail,

      },
      data() {
        return{
          description: '发票管理页面',
          isShow:false,
          // 查询条件
          queryParam: {},
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
              title: '合同名称',
              align: "center",
              dataIndex: 'contractName',
            },
            {
              title: '开票日期',
              align: "center",
              dataIndex: 'invociTime',
            },
            {
              title: '发票内容',
              align: "center",
              dataIndex: 'content',
            },
            {
              title: '开票金额',
              align: "center",
              dataIndex: 'totalMoney',
            },
            {
              title: '公司名称',
              align: "center",
              dataIndex: 'companyName',
            },
            {
              title: '税号',
              align: "center",
              dataIndex: 'shuihao',
            },
           {
              title: '附件',
              align: "center",
              dataIndex: 'fileCount',
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
          },
          loading: false,
          selectedRowKeys: [],
          selectedRows: [],
          url: {
            list: "/renche/invoci/qryInvociList",
            delete: "/renche/invoci/delete",
            deleteBatch: "/renche/invoci/deleteBat",
            filelist: "/renche/purchase/fileList",
            exportList: "/renche/invoci/exportInvoci",
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
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
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
        superQuery() {
          if(this.isShow){
            this.isShow = false;
          }else{
            this.isShow = true;
          }
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

              ids += this.selectionRows[a].invociId + ",";
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
        handleDelete: function (id) {
          var that = this;
          this.$confirm({
            title: "是否确认删除该数据?",
            onOk: function () {
              deleteAction(that.url.delete, {id: id}).then((res) => {
                if (res.success) {
                  that.$message.success(res.message);
                  that.loadData();
                } else {
                  that.$message.warning(res.message);
                }
              });
            }
          });
        },
        async handleEdit (record) {
          if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
            let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
            record.filelist = result.list;
          }
          this.$refs.addInvociInfo.edit(record);
          this.$refs.addInvociInfo.title = "编辑";
        },

        fileDeteil:function(record){
          console.log(record);
          this.$refs.invociInfoFileDetail.fileLoad(record);
          this.$refs.invociInfoFileDetail.title = "附件";
        },

        handleAdd: function () {
          this.$refs.addInvociInfo.add();
          this.$refs.addInvociInfo.title = "新增";
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
        },
        modalFormOk() {
          // 新增/修改 成功时，重载列表
          this.loadData(1);
        },
        exportDate(){
          var params = Object.assign({}, this.queryParam, this.isorter);
          var param = JSON.stringify(params);
          param = param.replace("{","");
          param = param.replace("}","");
          window.location.href = window._CONFIG['domainURL'] + this.url.exportList + "?param="+param;
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