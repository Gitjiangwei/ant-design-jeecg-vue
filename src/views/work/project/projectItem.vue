<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="工程名称">
                <a-input placeholder="请输入工程名称" v-model="queryParam.prjItemName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="项目名称">
                <a-input placeholder="请输入项目名称" v-model="queryParam.prjName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="工程类型">
                <RencheDictSelectTag v-model="queryParam.prjItemType" placeholder="请输入工程类型" dictCode="PROJECTITEMTYPE"/>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="所属公司">
                <a-input placeholder="请输入所属公司名称" v-model="queryParam.belongCompany"></a-input>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24" v-show="isShow">
            <a-col :span="6">
              <a-form-item label="工程编号">
                <a-input placeholder="请输入工程编号" v-model="queryParam.prjItemNum"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="工程状态">
                <a-select placeholder="请选择工程状态" v-model="queryParam.prjItemStatus">
                  <a-select-option value="">请选择</a-select-option>
                  <a-select-option value="0">未开始</a-select-option>
                  <a-select-option value="1">进行中</a-select-option>
                  <a-select-option value="2">已结束</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="负责人">
                <a-input placeholder="请输入负责人" v-model="queryParam.personInCharge"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="工程进度">
                <a-input placeholder="请输入工程进度" v-model="queryParam.progressOfItem"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="进场时间">
                <a-date-picker placeholder="请输入进场时间" v-model="queryParam.entryTime"></a-date-picker>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="完成时间">
                <a-date-picker placeholder="请输入完成时间" v-model="queryParam.finishTime"></a-date-picker>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="要求部署时间">
                <a-date-picker placeholder="请输入要求部署时间" v-model="queryParam.requireDeployTime"></a-date-picker>
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

        <a-button @click="handleExport" type="primary" icon="download">下载导入模板</a-button>
        <a-upload name="file" :showUploadList="false" :multiple="false" :action="importExcelUrl" :beforeUpload="beforeUpload" accept=".xls">
          <a-button type="primary" icon="import">导入</a-button>
        </a-upload>
        <a-button @click="exportPrjItemDate" type="primary" icon="export">导出</a-button>
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
          rowKey="prjItemId"
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
                  <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.prjItemId)">
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
      <project-item-modal ref="projectItemModal" @ok="modalFormOk"></project-item-modal>

    </a-card>

</template>

<script>
    import ProjectItemModal from './modules/ProjectItemModal'
    import {filterObj} from '@/utils/util'
    import {deleteAction, getAction} from '@/api/manage'
    import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
    import ARow from "ant-design-vue/es/grid/Row";
    import moment from "moment"
    import { doMian} from '@/api/api'
    import axios from 'axios';

    export default {
      name: "projectItenList",
      components: {
        ARow,
        ProjectItemModal,
      },
      data() {
        return{
          description: '工程点管理页面',

          // 查询条件
          queryParam: {},
          //字典数组缓存
          typeDictOptions: [],
          isShow:false,
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
              title: '工程编号',
              align: "center",
              dataIndex: 'prjItemNum'
            },
            {
              title: '工程名称',
              align: "center",
              dataIndex: 'prjItemName'
            },
            {
              title: '工程类型',
              align: "center",
              dataIndex: 'prjItemType',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                return filterDictText(this.typeDictOptions, text);
              }
            },
            {
              title: '项目名称',
              align: "center",
              dataIndex: 'prjName'
            },
            {
              title: '所属公司',
              align: "center",
              dataIndex: 'companyName'
            },
            {
              title: '负责人',
              align: "center",
              dataIndex: 'personInCharge'
            },
            {
              title: '工程状态',
              align: "center",
              dataIndex: 'prjItemStatus',
              customRender: (text, record, index) => {
                if(text == '0'){
                  return "未开始";
                }else if(text == '1'){
                  return "进行中";
                }else if(text == '2'){
                  return "已结束";
                }
              }
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
          fileList: [],
          loading: false,
          selectedRowKeys: [],
          selectedRows: [],
          url: {
            list: "/renche/projectItem/list",
            delete: "/renche/projectItem/delete",
            deleteBatch: "/renche/projectItem/deleteBatch",
            importExcelUrl: doMian + "/renche/projectItem/importPrjItem"
          },
        }
      },
      computed: {
        importExcelUrl: function(){
          const { fileList } = this;
          const formData = new FormData();
          if(fileList.length > 0){
            fileList.forEach((file) => {
              formData.append('file', file);
            });
            axios({/*192.168.133.15:8086*/
              url: this.url.importExcelUrl,
              method: 'post',
              processData: false,
              data: formData
            }).then((res) => {
              if(res.data.message == ""){
                this.$message.warning("导入成功");
              }else{
                this.$message.warning(res.data.message);
              }
              this.fileList = [];
              this.loadData();
            })
          }
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
          if(params.entryTime != undefined){
            var entryTime = moment(params.entryTime).format('YYYY-MM-DD');
            params.entryTime = entryTime;
          }
          if(params.finishTime != undefined){
            var finishTime = moment(params.finishTime).format('YYYY-MM-DD');
            params.finishTime = finishTime;
          }
          if(params.requireDeployTime != undefined){
            var requireDeployTime = moment(params.requireDeployTime).format('YYYY-MM-DD');
            params.requireDeployTime = requireDeployTime;
          }
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
        },
        initDictConfig() {
          //初始化字典 - 工程类型
          initDictOptions('PROJECTITEMTYPE').then((res) => {
            if (res.success) {
              this.typeDictOptions = res.result;
            }
          });
        },
        handleSuperQuery(arg) {//高级查询方法
          let params = {'superQueryParams':encodeURI(JSON.stringify(arg))};
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }else{
              that.$message.warn(res.message);
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
        handleDelete: function (id) {
          var that = this;
          deleteAction(that.url.delete, {id: id}).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.loadData();
            } else {
              that.$message.warning(res.message);
            }
          });
        },
        handleEdit: function (record) {
          this.$refs.projectItemModal.edit(record);
          this.$refs.projectItemModal.title = "编辑";
        },
        handleAdd: function () {
          this.$refs.projectItemModal.add();
          this.$refs.projectItemModal.title = "新增";
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
        handleExport(){
         var href= 'http://localhost:3000/jeecg-boot/renche/projectItem/exportPrjItemModel';
         window.location = href;
        },
        beforeUpload(file) {
          this.fileList = [...this.fileList, file];
          return false;
        },
        exportPrjItemDate(){
          //加载数据 若传入参数1则加载第一页的内容
          if (arg === 1) {
            this.ipagination.current = 1;
          }
          var params = this.getQueryParams();//查询条件
          if(params.entryTime != undefined){
            var entryTime = moment(params.entryTime).format('YYYY-MM-DD');
            params.entryTime = entryTime;
          }
          if(params.finishTime != undefined){
            var finishTime = moment(params.finishTime).format('YYYY-MM-DD');
            params.finishTime = finishTime;
          }
          if(params.requireDeployTime != undefined){
            var requireDeployTime = moment(params.requireDeployTime).format('YYYY-MM-DD');
            params.requireDeployTime = requireDeployTime;
          }
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
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