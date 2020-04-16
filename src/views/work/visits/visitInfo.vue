<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="客户名称" >
                <a-input placeholder="请输入客户名称" v-model="queryParam.companyName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="工单名称">
                <a-input placeholder="请输入工单名称" v-model="queryParam.workName"></a-input>
              </a-form-item>
            </a-col>

          <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>


            </a-col>
          </a-row>
        </a-form>
      </div>

      <!-- 操作按钮区域 -->
      <div class="table-operator">
       <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
       <!-- <a-button type="primary" >
          <a :href="'http://localhost:8080/jeecg-boot/renche/visit/exportVisit1'" target="_blank" style="margin-left: 10px">导出</a>
        </a-button>-->
        <a-button @click="exportDate" type="primary" icon="export">导出</a-button>

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
          rowKey="id"
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
            <a-dropdown>
              <!--<a class="ant-dropdown-link">更多 <a-icon type="down"/></a>-->
              <a-menu slot="overlay">
                <a-menu-item>
                  <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.visitId )">
                    <a>删除</a>
                  </a-popconfirm>
                </a-menu-item>
              </a-menu>
            </a-dropdown>
          </span>
        </a-table>
      </div>
      <!-- table区域-end -->

      <!-- 表单区域 -->

      <add-visit ref="addVisit" @ok="modalFormOk"></add-visit>
      <file-detail ref="fileDetail" @ok="modalFormOk"></file-detail>

    </a-card>

</template>

<script>
    import ARow from "ant-design-vue/es/grid/Row";
    import addVisit from './modules/addVisitInfo';
    import {filterObj} from '@/utils/util';
    import fileDetail from "./modules/FileDetail";
    import {deleteAction, getAction, postAction} from '@/api/manage';
    import {initDictOptions} from '@/components/dict/RencheDictSelectUtil';


    export default {
      name: "visitInfo",
      components: {
        ARow,
        addVisit,
        fileDetail,
      },
      data() {
        return{
          description: '我的工单',
          fileRelId:"",
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
              title: '工单名称',
              align: "center",
              dataIndex: 'workName',
            },
            {
              title: '客户名称',
              align: "center",
              dataIndex: 'companyName',
            },
            {
              title: '客户电话',
              align: "center",
              dataIndex: 'phone',
            },
           /* {
              title: '拜访人',
              align: "center",
              dataIndex: 'visitor',
            },*/
          /*  {
              title: '拜访时间',
              align: "center",
              dataIndex: 'visitTime',
            },*/

            {
              title: '计划执行时间',
              align: "center",
              dataIndex: 'planExecuTime',

            },


            {
              title: '实际执行时间',
              align: "center",
              dataIndex: 'realityExecuTime',

            },
            {
              title: '计划参与人数',
              align: "center",
              dataIndex: 'planPersonNum',

            },
            {
              title: '实际参与人数',
              align: "center",
              dataIndex: 'realityPersonNum',

            },
            {
              title: '任务内容',
              align: "center",
              dataIndex: 'content',
            },
            {
              title: '执行情况',
              align: "center",
              dataIndex: 'result',

            },
            {
              title: '用户评价',
              align: "center",
              dataIndex: 'evaluate',

            },
            {
              title: '备注',
              align: "center",
              dataIndex: 'remark',

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
            list: "/renche/WorkSerivice/qryWorkSerivice",
            delete: "/renche/visit/delete",
            deleteBatch:"/renche/visit/deleteBatch",
            export:"/renche/WorkSerivice/exportWorkService",

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
        handleDelete: function (id) {
          var that = this;
          postAction(that.url.delete, {id: id}).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
             /* alert("已删除");*/
              that.loadData();
            } else {
              that.$message.warning(res.message);
            }
          });
        },


        exportfile: function () {
          getAction(this.url.export).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
        },
         handleEdit: function (record) {
          this.$refs.addVisit.edit(record);
          this.$refs.addVisit.title = "编辑";
        },


        fileDeteil:function(record){
          console.log(record);
          this.$refs.fileDetail.fileLoad(record);
          this.$refs.fileDetail.title = "附件";
        },
        handleAdd: function () {
          this.$refs.addVisit.add();
          this.$refs.addVisit.title = "新增";
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
          this.loadData();
        },
        exportDate(){
         var params = Object.assign({}, this.queryParam, this.isorter);
          var param = JSON.stringify(params);
          //alert("param="+param);
          param = param.replace("{","");
          param = param.replace("}","");
          window.location.href = window._CONFIG['domainURL'] + this.url.export + "?param="+param;
        }
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