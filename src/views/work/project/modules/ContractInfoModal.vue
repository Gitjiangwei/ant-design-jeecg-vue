<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="合同名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入合同名称" v-decorator="['contractName', {rules: [{ required: true, message: '请输入合同名称', }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="甲方公司" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入甲方公司"  v-decorator="['companyNameA', {rules: [{ required: true, message: '请输入甲方公司', }]}]"  @blur="checkThisCompanyIsExsit" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="甲方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入甲方合同编号" v-decorator="['contractNoA', {rules: [{ required: true, message: '请输入甲方合同编号', }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="乙方公司" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入乙方公司" v-decorator="['companyNameB', {rules: [{ required: true, message: '请输入乙方公司', }]}]"  @blur="checkThisCompanyIsExsit" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="乙方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入乙方合同编号" v-decorator="['contractNoB', {rules: [{ required: true, message: '请输入乙方合同编号', }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="合同类型" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-select v-decorator="['contractType', {rules: [{ required: true, message: '请选择合同类型', }]}]" placeholder="请选择合同类型">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="合同金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
                <a-input placeholder="请输入合同金额(万元)" v-decorator="['contractMoney', {rules:[{pattern: /^-?\d+\.?\d{0,2}$/, message: '请输入正确数据', trigger: 'blur'}]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="要求部署时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker placeholder="请输入要求部署时间" v-decorator="['requireDeployTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="签订状态" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-select v-decorator="['contractStatus', {rules: [{ required: true, message: '请选择合同签订状态', }]}]" placeholder="请选择合同签订状态">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option value="0">未签订</a-select-option>
                <a-select-option value="1">已签订</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="签收时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker placeholder="请输入签收时间" v-decorator="['signInTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row v-show="isEdit">
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="回款金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['allReturnMoney', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="回款占比" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['returnMoneyPercent', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="电子版附件"
              hasFeedback>
              <!--  -->
              <a-upload
                name="file"
                :multiple="true"
                :action="uploadAction"
                :headers="headers"
                @change="handleChangeElec"
              >
                <a-button>
                  <a-icon type="upload"/>
                  上传
                </a-button>
              </a-upload>
              <div>
                <div v-for="(item,index) in model.filelist" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="扫描件附件"
              :isShowUploadList="isEdit"
              hasFeedback>
              <!--  -->
              <a-upload
                name="file"
                :multiple="true"
                :action="uploadAction"
                :headers="headers"
                @change="handleChange"
              >
                <a-button>
                  <a-icon type="upload"/>
                  上传
                </a-button>
              </a-upload>
              <div>
                <div v-for="(item,index) in model.filelist" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="提醒周期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-select v-decorator="['remindPeriodType', {rules: [{ required: true, message: '请选择提醒周期类型', }]}]" placeholder="请选择提醒周期类型">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option v-for="item in remindTypeOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
              </a-select>
              <a-input placeholder="请输入周期长度" v-decorator="['remindPeriod', {rules: [{ pattern: /^\d{0,2}$/, message: '请选择提醒周期长度', }]}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="关联招标" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入招标信息" v-decorator="['prjName', {}]" @click="showTender"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" v-show="isEdit" >
      <a-tab-pane tab="关联工程点" key="1" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
        <div style="float: right;">
          <a-button @click="showProjectItem" type="primary" icon="plus">新增</a-button>
        </div>

        <div style="padding-top: 42px;">
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="prjItemId"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :loading="loading"
            @change="handleTableChange" style="padding-top: 10px;">

            <span slot="action" slot-scope="text, record">
              <a @click="handleDel(record.prjItemId)">删除</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>

      <a-tab-pane tab="开票信息" key="2" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
        <div style="float: right;">
          <a-button @click="addInvoiceInfo" type="primary" icon="plus">新增</a-button>
        </div>

        <div style="padding-top: 42px;">
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="invoiceId"
            :columns="invoiceColumns"
            :dataSource="invoiceDataSource"
            :pagination="invoiceIpagination"
            :loading="invoiceLoading"
            @change="handleTableChangeInvoice" style="padding-top: 10px;">

            <span slot="action" slot-scope="text, record">
              <a @click="fileDeteil(record)">附件</a>
              <a-divider type="vertical"/>
              <a @click="handleEdit(record)">编辑</a>
              <a-divider type="vertical"/>
              <a @click="handleDelete(record.invoiceId)">删除</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>
    </a-tabs>

    <TenderInfoShow ref="tenderInfoShow" @func="modalFormOk"></TenderInfoShow>

    <project-item-show  ref="projectItemShow" @func="addProjectItem"></project-item-show>

    <InvoiceInfoModel  ref="invoiceInfoModel" @func="invoiceInfoShowList"></InvoiceInfoModel>

    <file-detail ref="fileDetail" @func="invoiceInfoShowList"></file-detail>
  </a-modal>
</template>

<script>
  import TenderInfoShow from './TenderInfoShow'
  import { httpAction, getAction, deleteAction} from '@/api/manage'
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import {filterObj} from '@/utils/util'
  import ARow from "ant-design-vue/es/grid/Row";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ACol from "ant-design-vue/es/grid/Col";
  import ProjectItemShow from "./ProjectItemShow";
  import InvoiceInfoModel from "./InvoiceInfoModel";
  import { doMian} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import fileDetail from "./FileDetail";

  export default {
    name: "projectItemModal",
    components: {ProjectItemShow, ACol, ATextarea, ARow,TenderInfoShow,InvoiceInfoModel,fileDetail},
    data () {
      return {
        title:"操作",
        visible: false,
        isEdit: false,
        model: {},
        //字典数组缓存
        typeDictOptions: [],
        remindTypeOptions: [],
        typePrjDictOptions: [],
        isOk:true,
        companyIdA:"",
        companyIdB:"",
        thisMessage:"",
        defaultActiveKey: '1',
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        uploadLoading: false,
        headers: {},
        avatar: "",
        avatarElec: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        // 工程点表头
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
              return filterDictText(this.typePrjDictOptions, text);
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
          pageSize: 5,
          pageSizeOptions: ['5', '10', '15'],
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

        // 开票信息表头
        invoiceColumns: [
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
            title: '开票时间',
            align: "center",
            dataIndex: 'invoiceTime'
          },
          {
            title: '开票金额(万元)',
            align: "center",
            dataIndex: 'invoiceMoney'
          },
          {
            title: '回款时间',
            align: "center",
            dataIndex: 'returnTime'
          },
          {
            title: '回款金额(万元)',
            align: "center",
            dataIndex: 'returnMoney'
          },
          {
            title: '附件',
            align: "center",
            dataIndex: 'fileRelId',
            customRender: (text) => {
              if(text!=null && text !="" && text != undefined) {
                if(text.indexOf(",") != -1) {
                  var count = text.match(/,/g).length;
                  return parseInt(count);
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
        invoiceDataSource: [],
        // 分页参数
        invoiceIpagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '15'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        invoiceIsorter: {
          column: 'createTime',
          order: 'desc',
        },
        invoiceLoading: false,
        invoiceSelectedRowKeys: [],
        invoiceSelectedRows: [],
        url: {
          add: "/renche/contractInfo/add",
          edit: "/renche/contractInfo/edit",
          checkCompany: "/renche/companyInfo/checkNameIsExsit",
          prjItemList: "/renche/contractInfo/contractPrjItemList",
          delprjItem: "/renche/contractInfo/delPrjItem",
          invoiceList: "/renche/invoiceInfo/contractInvoiceList",
          delInvoiceInfo: "/renche/invoiceInfo/delete",
          fileUpload: doMian + "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
      //初始化字典配置
      this.initDictConfig();
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 合同类型
        initDictOptions('CONTRACTTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
        //初始化字典 - 合同提醒周期类型
        initDictOptions('REMINDPERIODTYPE').then((res) => {
          if (res.success) {
            this.remindTypeOptions = res.result;
          }
        });
        //初始化字典 - 工程类型
        initDictOptions('PROJECTITEMTYPE').then((res) => {
          if (res.success) {
            this.typePrjDictOptions = res.result;
          }
        });
      },
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.prjItemList, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      loadInvoiceData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.invoiceIpagination.current = 1;
        }
        getAction(this.url.invoiceList, {pageNo: this.invoiceIpagination.current,pageSize: this.invoiceIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
          if (res.success) {
            this.invoiceDataSource = res.result.list;
            this.invoiceIpagination.total = res.result.total;
          }
        })
      },
      getQueryParams() {
        var param = {};
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        param.contractId = this.model.contractId;
        return filterObj(param);
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.avatar = record.fileRelId;
        this.avatarElec = record.elecFileRel;

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.isEdit = false;
        this.dataSource = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'contractName','companyNameA','contractNoA','companyNameB','contractNoB','contractType','contractMoney','contractStatus','remindPeriod','prjName','remark'))
        });

        this.form.setFieldsValue({requireDeployTime:this.model.requireDeployTime?moment(this.model.requireDeployTime):null});
        this.form.setFieldsValue({signInTime:this.model.signInTime?moment(this.model.signInTime):null});

        if(this.model.contractId != null && this.model.contractId != undefined){
          this.isEdit = true;
          getAction(this.url.prjItemList, {pageNo: this.ipagination.current,pageSize: this.ipagination.pageSize,contractId: this.model.contractId}).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
          getAction(this.url.invoiceList, {pageNo: this.invoiceIpagination.current,pageSize: this.invoiceIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
            if (res.success) {
              this.invoiceDataSource = res.result.list;
              this.invoiceIpagination.total = res.result.total;
            }
          })
        }
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            if(that.isOk){
              that.confirmLoading = true;
              let httpurl = '';
              let method = '';
              if(!that.model.contractId){
                httpurl+=this.url.add;
                method = 'post';
              }else{
                httpurl+=this.url.edit;
                method = 'put';
              }
              if(that.companyIdA != ''){
                this.model.partyA = that.companyIdA;
              }
              if(that.companyIdB != ''){
                this.model.partyB = that.companyIdB;
              }

              this.model.fileRelId = that.avatar;
              this.model.elecFileRel = that.avatarElec;

              let formData = Object.assign(this.model, values);

              httpAction(httpurl,formData,method).then((res)=>{
                if(res.success){
                  that.$message.success(res.message);
                  that.$emit('ok');
                }else{
                  that.$message.warning(res.message);
                }
              }).finally(() => {
                that.confirmLoading = false;
                that.close();
              })
            }else{
              that.$message.warning(this.thisMessage);
              that.thisMessage = "";
            }
          }
        })
      },
      handleCancel () {
        this.close()
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
      handleTableChangeInvoice(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.invoiceIsorter.column = sorter.field;
          this.invoiceIsorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadInvoiceData();
      },
      checkThisCompanyIsExsit(e){
        var id = e.target.id;
        var companyName = e.target.value;//查询条件
        if(companyName != null && companyName != "") {
          getAction(this.url.checkCompany, {name: companyName}).then((res) => {
            if (res.success) {
              this.isOk = true;
              if(id == 'companyNameA'){
                this.companyIdA = res.message;
              }
              if(id == 'companyNameB'){
                this.companyIdB = res.message;
              }

            } else {
              this.isOk = false;
              if (res.code = '200') {
                this.$message.warning(res.message);
              }
            }
          })
        }
      },
      showTender:function (){
        this.$refs.tenderInfoShow.show();
        this.$refs.tenderInfoShow.title = "选择关联招标信息";
      },
      modalFormOk(data) {
        this.model.tenderId = data[0].tenderId;
        this.form.setFieldsValue({prjName:data[0].prjName});
      },
      callback: function(key){
          this.defaultActiveKey = key;
      },
      showProjectItem:function (){
        this.$refs.projectItemShow.show(this.model.contractId);
        this.$refs.projectItemShow.title = "选择添加关联工程点信息";
      },
      addProjectItem(data) {
        this.loadData();
      },
      handleDel: function (id) {
        var that = this;
        var contractId = this.model.contractId;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delprjItem, {prjItemId: id, contractId: contractId}).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.loadData();
              } else {
                that.$message.warning(res.message);
              }
            });
          }
        })
      },
      addInvoiceInfo:function (){
        var record = {contractId: this.model.contractId};
        this.$refs.invoiceInfoModel.edit(record);
        this.$refs.invoiceInfoModel.title = "新增开票信息";
      },
      invoiceInfoShowList(data) {
        this.loadInvoiceData();
      },
      handleEdit:function (record){
        this.$refs.invoiceInfoModel.edit(record);
        this.$refs.invoiceInfoModel.title = "编辑开票信息";
      },
      handleDelete: function (id) {
        var that = this;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delInvoiceInfo, {id: id}).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.loadInvoiceData();
              } else {
                that.$message.warning(res.message);
              }
            });
          }
        })
      },
      handleChangeElec(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return;
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatarElec = response.message + "," + this.avatarElec;
            console.log(this.avatarElec);
          } else {
            this.$message.warning(response.message);
          }
        }
      },
      handleChange(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return;
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatar = response.message + "," + this.avatar;
            console.log(this.avatar);
          } else {
            this.$message.warning(response.message);
          }
        }
      },
      fileDeteil:function(record){
        console.log(record);
        this.$refs.fileDetail.fileLoad(record);
        this.$refs.fileDetail.title = "附件";
      },
    }
  }
</script>

<style scoped>

</style>