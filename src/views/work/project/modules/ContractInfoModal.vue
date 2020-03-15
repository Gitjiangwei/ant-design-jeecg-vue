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
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="提醒周期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-select v-decorator="['remindPeriod', {}]" placeholder="请选择提醒周期">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option value="0">未签订</a-select-option>
                <a-select-option value="1">已签订</a-select-option>
              </a-select>
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

    <a-tabs type="card" v-show="isEdit">
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
    </a-tabs>

    <TenderInfoShow ref="tenderInfoShow" @func="modalFormOk"></TenderInfoShow>

    <project-item-show  ref="projectItemShow" @func="addProjectItem"></project-item-show>
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

  export default {
    name: "projectItemModal",
    components: {ProjectItemShow, ACol, ATextarea, ARow,TenderInfoShow},
    data () {
      return {
        title:"操作",
        visible: false,
        isEdit: false,
        model: {},
        //字典数组缓存
        typeDictOptions: [],
        typePrjDictOptions: [],
        isOk:true,
        companyIdA:"",
        companyIdB:"",
        thisMessage:"",
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
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
          add: "/renche/contractInfo/add",
          edit: "/renche/contractInfo/edit",
          checkCompany: "/renche/companyInfo/checkNameIsExsit",
          prjItemList: "/renche/contractInfo/contractPrjItemList",
          delprjItem: "/renche/contractInfo/delPrjItem"
        },
      }
    },
    created () {
      //初始化字典配置
      this.initDictConfig();
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 合同类型
        initDictOptions('CONTRACTTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
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
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
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
      showProjectItem:function (){
        this.$refs.projectItemShow.show(this.model.contractId);
        this.$refs.projectItemShow.title = "选择添加关联工程点信息";
      },
      addProjectItem(data) {
        this.loadData();
      },
      handleDel: function (id) {
        var that = this;
        deleteAction(that.url.delprjItem, {prjItemId: id, contractId: this.model.contractId}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },

    }
  }
</script>

<style scoped>

</style>