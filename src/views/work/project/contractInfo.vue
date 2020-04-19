<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="合同名称">
                <a-input placeholder="请输入合同名称" v-model="queryParam.contractName" maxlength="150"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="合同类型">
                <RencheDictSelectTag v-model="queryParam.contractType" placeholder="请选择合同类型" dictCode="CONTRACTTYPE"/>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="甲方公司">
                <a-input placeholder="请输入甲方公司名称" v-model="queryParam.partyA"  maxlength="30"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="乙方公司">
                <a-input placeholder="请输入乙方公司名称" v-model="queryParam.partyB"  maxlength="30"></a-input>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row  :gutter="24" v-show="isShow">
            <a-col :span="6"  >
              <a-form-item label="合同签订状态">
                <a-select placeholder="请选择合同签订状态" v-model="queryParam.contractStatus">
                  <a-select-option value="">请选择</a-select-option>
                  <a-select-option value="2">未签订</a-select-option>
                  <a-select-option value="1">已签订</a-select-option>
                  <a-select-option value="0">已结束</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="甲方合同编号">
                <a-input placeholder="请输入甲方合同编号" v-model="queryParam.contractNoA"  maxlength="30"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="乙方合同编号">
                <a-input placeholder="请输入乙方合同编号" v-model="queryParam.contractNoB"  maxlength="30"></a-input>
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
                  <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.contractId)">
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
      <contract-info-modal ref="contractInfoModal" @ok="modalFormOk"></contract-info-modal>

      <contract-file-detail ref="contractFileDetail" @ok="modalFormOk"></contract-file-detail>
    </a-card>

</template>

<script>
    import ContractInfoModal from './modules/ContractInfoModal'
    import {filterObj} from '@/utils/util'
    import {deleteAction, getAction} from '@/api/manage'
    import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
    import ARow from "ant-design-vue/es/grid/Row";
    import ContractFileDetail from "./modules/ContractFileDetail";

    export default {
      name: "contractInfoList",
      components: {
        ARow,
        ContractInfoModal,
        ContractFileDetail,
      },
      data() {
        return{
          description: '合同信息管理页面',

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
              title: '合同名称',
              align: "center",
              dataIndex: 'contractName'
            },
            {
              title: '合同类型',
              align: "center",
              dataIndex: 'contractType',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                return filterDictText(this.typeDictOptions, text);
              }
            },
            {
              title: '甲方公司',
              align: "center",
              dataIndex: 'companyNameA'
            },
            {
              title: '乙方公司',
              align: "center",
              dataIndex: 'companyNameB'
            },
            {
              title: '合同金额(万元)',
              align: "center",
              dataIndex: 'contractMoney'
            },
            {
              title: '合同状态',
              align: "center",
              dataIndex: 'contractStatus',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                if(text == '2'){
                  return "未签订";
                }else if (text == '1'){
                  return "已签订";
                }else if (text == '0'){
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
            list: "/renche/contractInfo/list",
            delete: "/renche/contractInfo/delete",
            deleteBatch: "/renche/contractInfo/deleteBatch",
            filelist: "/renche/purchase/fileList",
            exportList: "/renche/contractInfo/exportContractInfo"
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
        async handleEdit(record) {
          if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
            let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
            record.filelist = result.list;
          }
          if(record.elecFileRel != null || record.elecFileRel != "" || record.elecFileRel != undefined) {
            let {result} = await getAction(this.url.filelist, {fileRelId: record.elecFileRel});
            record.elefilelist = result.list;
          }
          this.$refs.contractInfoModal.edit(record);
          this.$refs.contractInfoModal.title = "编辑";
        },
        fileDeteil:function(record){
          console.log(record);
          this.$refs.contractFileDetail.fileLoad(record);
          this.$refs.contractFileDetail.title = "附件";
        },
        handleAdd: function () {
          this.$refs.contractInfoModal.add();
          this.$refs.contractInfoModal.title = "新增";
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