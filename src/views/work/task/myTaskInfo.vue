<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="任务名称">
                <a-input placeholder="请输入任务名称" v-model="queryParam.taskName" maxlength="150"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="工程名称">
                <a-input placeholder="请输入工程名称" v-model="queryParam.prjItemName" maxlength="150"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="任务状态">
                <a-select v-model="queryParam.status"  placeholder="请选择任务状态">
                  <a-select-option value="0">新建</a-select-option>
                  <a-select-option value="1">进行中</a-select-option>
                  <a-select-option value="2">已结束</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="创建人">
                <a-input placeholder="请输入创建人" v-model="queryParam.createUserName"  maxlength="30"></a-input>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24" v-show="isShow">
            <a-col :span="12"  >
              <a-form-item label="开始时间">
                <a-date-picker v-decorator="[ 'startTime', {}]" />  --  <a-date-picker v-decorator="[ 'startTime', {}]" />
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24">
            <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>
                <a-button @click="exportDate" type="primary" icon="export">导出</a-button>
                <a-button v-if="selectedRowKeys.length > 0" @click="batchMakeDemand" type="primary" icon="plus">批量生成设备需求</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
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
          rowKey="contractId"
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
              <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
              <a-menu slot="overlay">
                <a-menu-item>
                  <a-popconfirm title="确定生成设备需求?" @confirm="() => handleMakeDemand(record)">
                    <a>生成需求</a>
                  </a-popconfirm>
                </a-menu-item>
                <a-menu-item>
                  <a-popconfirm title="确定结束任务吗?" @confirm="() => handleEnd(record.taskId, '2')">
                    <a>结束</a>
                  </a-popconfirm>
                </a-menu-item>
              </a-menu>
            </a-dropdown>
          </span>
        </a-table>
      </div>
      <!-- table区域-end -->

      <!-- 表单区域 -->
      <MyTaskInfoModel ref="taskInfoModel" @ok="modalFormOk"></MyTaskInfoModel>

      <contract-file-detail ref="contractFileDetail" @ok="modalFormOk"></contract-file-detail>
    </a-card>

</template>

<script>
    import MyTaskInfoModel from './modules/MyTaskInfoModel'
    import {filterObj} from '@/utils/util'
    import {deleteAction, getAction} from '@/api/manage'
    import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
    import ARow from "ant-design-vue/es/grid/Row";
    import ContractFileDetail from "../project/modules/ContractFileDetail";
    import moment from "moment"

    export default {
      name: "myTaskInfoList",
      components: {
        ARow,
        MyTaskInfoModel,
        ContractFileDetail,
      },
      data() {
        return{
          // 查询条件
          queryParam: {},
          //字典数组缓存
          typeDictOptions: [],
          isShow:false,
          receiveUser: '',
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
              title: '任务名称',
              align: "center",
              dataIndex: 'taskName'
            },
            {
              title: '工程名称',
              align: "center",
              dataIndex: 'prjItemName'
            },
            {
              title: '计划开始时间',
              align: "center",
              dataIndex: 'planStartTime'
            },
            {
              title: '实际开始时间',
              align: "center",
              dataIndex: 'startTime'
            },
            {
              title: '负责人',
              align: "center",
              dataIndex: 'receiveUserName'
            },
            {
              title: '任务状态',
              align: "center",
              dataIndex: 'status',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                if (text == '0'){
                  return "新建";
                }else if(text == '1'){
                  return "进行中";
                }else if (text == '2'){
                  return "已结束";
                }
              }
            },
            {
              title: '附件',
              align: "center",
              dataIndex: 'fileCount'
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
            list: "/renche/taskInfo/qryMyTaskInfoList",
            filelist: "/renche/purchase/fileList",
            exportList: "/renche/taskInfo/exportTaskInfo",
            editTaskStatus: "/renche/taskInfo/editTaskStatus",
            makeDemandList: "/renche/taskInfo/makeDemandList",
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
          if(params.startTime != undefined){
            var startTime = moment(params.startTime).format('YYYY-MM-DD');
            params.startTime = startTime;
          }
          if(params.endTime != undefined){
            var endTime = moment(params.endTime).format('YYYY-MM-DD');
            params.endTime = endTime;
          }
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.receiveUser = res.message;
              this.ipagination.total = res.result.total;
            }
          })
        },
        initDictConfig() {
          //初始化字典 - 合同类型
          initDictOptions('CONTRACTTYPE').then((res) => {
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
        async handleEdit(record) {
          if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
            let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
            record.filelist = result.list;
          }
          this.$refs.taskInfoModel.edit(record);
          this.$refs.taskInfoModel.title = "编辑";
        },
        fileDeteil:function(record){
          console.log(record);
          this.$refs.contractFileDetail.fileLoad(record);
          this.$refs.contractFileDetail.title = "附件";
        },
        handleAdd: function () {
          this.$refs.taskInfoModel.add();
          this.$refs.taskInfoModel.title = "新增";
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
          params.mark = 'query';
          params.receiveUser = this.receiveUser;
          var param = JSON.stringify(params);
          param = param.replace("{","");
          param = param.replace("}","");
          window.location.href = window._CONFIG['domainURL'] + this.url.exportList + "?param="+param;
        },
        handleEnd(id, status){
          deleteAction(this.url.editTaskStatus, {taskId: id, status: status}).then((res)=>{
            if(res.success){
              that.$message.success(res.message);
              this.loadData();
            }else{
              that.$message.warning(res.message);
            }
          })
        },
        batchMakeDemand: function () {
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
              title: "确认生成设备需求",
              content: "是否确认生成设备需求",
              onOk: function () {
                deleteAction(that.url.makeDemandList, {taskIds: ids}).then((res) => {
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
        handleMakeDemand(record){
          var that = this;
          if(record.isMakeDemand == '1'){
            that.$message.warning("不能重复生成设备需求！");
          }else if(record.isMakeDemand == null || record.isMakeDemand == ''){
            that.$message.warning("请添加需要设备清单！");
          }else{
            this.$confirm({
              title: "确认生成设备需求",
              content: "再次确认是否生成设备需求",
              onOk: function () {
                deleteAction(that.url.makeDemandList, {taskIds: record.taskId}).then((res) => {
                  if (res.success) {
                    that.$message.success(res.message);
                    that.loadData();
                  } else {
                    that.$message.warning(res.message);
                  }
                });
              }
            });
          }
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