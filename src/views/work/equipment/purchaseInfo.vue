<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">

            <a-col :span="6">
              <a-form-item label="物品名称" >
                <a-input placeholder="请输入物品名称" maxlength="30" v-model="queryParam.purchaseItem"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="采购人">
                <a-input placeholder="请输入采购人" maxlength="20" v-model="queryParam.purchaser"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="是否到货">
                <a-select v-model="queryParam.isarrival">
                  <a-select-option :key="1">是</a-select-option>
                  <a-select-option :key="2">否</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="是否入库">
                <a-select v-model="queryParam.isstorage">
                  <a-select-option :key="1">已入库</a-select-option>
                  <a-select-option :key="2">未入库</a-select-option>
                </a-select>
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
      <div class="table-operator">
        <a-button @click="handleAdd" type="primary" icon="plus">新增 </a-button>
        <a-button @click="exportDate" type="primary" icon="export">导出</a-button>

       <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1" @click="batchDel(1)">
              <a-icon type="delete"/>
              删除
            </a-menu-item>
            <a-menu-item key="1" @click="batchDel(2)">
              <a-icon type="shopping-cart"/>
              收货
            </a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px"> 批量操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>
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


        <span slot="action" slot-scope="text, record">
          <a @click="fileDeteil(record)">附件</a>
           <a-divider type="vertical"/>
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a @click="handleDetail(record)">详情</a>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.purchaseId )">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定收货吗?" @confirm="() => batchDel(record.purchaseId)">
                  <a>收货</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定入库吗?程序会进行自动入库" @confirm="() => handleReceiving(record)">
                  <a>入库</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

        </a-table>
      </div>
      <!-- table区域-end -->
      <prochase-info-mode ref="prochaseInfoMode" @ok="modalFormOk"></prochase-info-mode>

      <pruchase-info-detail ref="pruchaseInfoDetail" ></pruchase-info-detail>

      <file-detail ref="fileDetail" @ok="modalFormOk"></file-detail>
    </a-card>

</template>

<script>
    import ARow from "ant-design-vue/es/grid/Row";
    import prochaseInfoMode from "./modules/pruchaseInfoMode";
    import pruchaseInfoDetail from "./modules/pruchaseInfoDetail";
    import fileDetail from "./modules/FileDetail";
    import {deleteAction, getAction, postAction} from '@/api/manage';
    import {filterObj,timeFix} from '@/utils/util';


    export default {
      name: "purchaseInfo",
      components: {
        ARow,
        prochaseInfoMode,
        pruchaseInfoDetail,
        fileDetail,
      },
      data() {
        return{
          description: '采购管理页面',
          timer:"",
          purchaseId:"",
          fileRelId:"",
          value:0,
          // 查询条件
          queryParam: {},
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
              title: '物品名称',
              align: "center",
              dataIndex: 'purchaseItem',
            },
            {
              title: '设备型号',
              align: "center",
              dataIndex: 'itemModel'
            },
            {
              title: '单价',
              align: "center",
              dataIndex: 'price'
            },
            {
              title: '数量',
              align: "center",
              dataIndex: 'quantity',

            },
            {
              title: '总价',
              align: "center",
              dataIndex: 'totalPrice',
            },
            {
              title: '采购人员',
              align: "center",
              dataIndex: 'purchaser'
            },
            {
              title: '采购时间',
              align: "center",
              dataIndex: 'purchaseTime'
            },
            {
              title: '采购来源',
              align: "center",
              dataIndex: 'whichCompany'
            },
            {
              title: '到货日期',
              align: "center",
              dataIndex: 'arrivalTime'
            },
            {
              title: '是否到货',
              align: "center",
              dataIndex: 'isarrival',
              customRender: (text) => {
                if(text==1){
                  return "是";
                }else if(text==2){
                  return "否";
                }else{
                  return text;
                }
              }
            },
            {
              title: '是否入库',
              align: "center",
              dataIndex: 'isstorage',
              customRender: (text) => {
                if(text==1){
                  return "已入库";
                }else if(text==2){
                  return "未入库";
                }else{
                  return text;
                }
              }
            },
            {
              title: '附件',
              align: "center",
              dataIndex: 'fileRelId',
              customRender: (text) => {
                if(text!=null && text !="" && text != undefined) {
                  if(text.indexOf(",") != -1) {
                    var count = text.match(/,/g).length;
                    return parseInt(count) + 1;
                  }else{
                    return 1;
                  }
                }else{
                  return 0;
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
            list: "/renche/purchase/qryPurchase",
            delete: "/renche/purchase/delete",
            deleteBatch: "/renche/purchase/deleteBatch",
            updateIsArrival: "/renche/purchase/updateIsArrival",
            receiningGoods: "/renche/purchase/insertReceiving",
            qryReceivingKey:"/renche/purchase/qryPurchaseKey",
            filelist: "/renche/purchase/fileList",
            export:"/renche/purchase/exportPurchase",
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

        fileDeteil:function(record){
            console.log(record);
            this.$refs.fileDetail.fileLoad(record);
            this.$refs.fileDetail.title = "附件";
        },
        handleAdd: function () {
           this.$refs.prochaseInfoMode.add();
          this.$refs.prochaseInfoMode.title = "新增";
        },
       async handleEdit  (record) {
        if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
          record.filelist = result.list;
        }
          //let results = this.handleKey(record);
          this.$refs.prochaseInfoMode.edit(record);
          this.$refs.prochaseInfoMode.title = "编辑";
        },
        handleDetail: function(record){
          this.$refs.pruchaseInfoDetail.detail(record);
          this.$refs.pruchaseInfoDetail.title = "详情";
        },
        batchDel: function (flag) {
          if (this.selectedRowKeys.length <= 0 && (flag==1||flag==2)) {
            this.$message.warning('请选择一条记录！');
            return;
          }else {
            var ids = "";
            if(flag==1||flag==2) {
              for (var a = 0; a < this.selectedRowKeys.length; a++) {
                ids += this.selectionRows[a].purchaseId + ",";
              }
            }else{
              ids = flag;
            }
            var that = this;
            var title = "";
            var content = "";
            var url = "";
            if (flag==1){
              title = "确认删除";
              content = "是否删除选中数据";
              url = that.url.deleteBatch;
            } else {
              title = "确认收货";
              content = "再次确认设备已经到达";
              url = that.url.updateIsArrival;
            }
            this.$confirm({
              title: title,
              content: content,
              onOk: function () {
                deleteAction(url, {ids: ids}).then((res) => {
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
        timelog(purchaseId){
          var that = this;
          this.value++;
          if(this.value==10){
            that.beforeDestroy();
            that.ReceivingError();
          }
          getAction(that.url.qryReceivingKey, {purchaseId:this.purchaseId}).then((res) => {
            if (res.success) {
              that.loadData();
              that.beforeDestroy();
              that.ReceivingSuccess();
            }

          });

        },
        mounted(){
          // 设置用间隔时间3s，每隔三秒调用一次。
          if(this.value==10){
            var that = this;
            that.beforeDestroy();
            that.ReceivingError();
          }
          if(this.purchaseId!=""){
            this.value++;
            this.timer = setInterval(this.timelog,3000);
          }
        },
        handleReceiving: function (record) {
          var that = this;
          debugger;
          if(record.isarrival==2){
            that.$message.warning("只能入库收货后的设备");
            return;
          }
          if(record.isstorage==1){
            that.$message.warning("当前的设备已经入库，不能重放入库！");
            return;
          }
          postAction(that.url.receiningGoods, {purchaseItem: record.purchaseItem,itemModel:record.itemModel,price:record.price,
            quantity:record.quantity,purchaseId:record.purchaseId}).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              this.purchaseId = record.purchaseId;
              that.mounted(record.purchaseId);
            } else {
              that.$message.warning(res.message);
            }
          });
        },
        //定时任务销毁
        beforeDestroy()
        {
          clearInterval(this.timer)
        },
        ReceivingSuccess () {
          //this.$router.push({ name: "dashboard" })
          this.$notification.success({
            message: '成功',
            description: `设备入库成功，请查收！`,
          });
        },
        ReceivingError () {
          //this.$router.push({ name: "dashboard" })
          this.$notification.success({
            message: '失败！',
            description: `设备入库失败，请及时排查！`,
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

</style>