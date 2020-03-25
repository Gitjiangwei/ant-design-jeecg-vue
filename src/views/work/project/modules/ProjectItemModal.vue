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
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程名称" >
              <a-input placeholder="请输入工程名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入工程名称', }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目名称" >
              <a-input placeholder="请输入项目名称" v-decorator="['prjName', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程编号" >
              <a-input placeholder="请输入工程编号" v-decorator="['prjItemNum', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程类型" >
              <a-select v-decorator="['prjItemType', {rules: [{ required: true, message: '请选择工程类型', }]}]" placeholder="请选择工程类型">
                <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
              <a-input placeholder="请输入所属公司" v-decorator="['companyName', {}]" @blur="checkThisCompanyIsExsit"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="负责人" >
              <a-input placeholder="请输入负责人" v-decorator="['personInCharge', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="联系电话" >
              <a-input placeholder="请输入联系电话" v-decorator="['personTel', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程状态" >
              <a-select v-decorator="['prjItemStatus', {rules: [{ required: true, message: '请选择工程状态', }]}]" placeholder="请选择工程状态">
                <a-select-option value="0">未开始</a-select-option>
                <a-select-option value="1">进行中</a-select-option>
                <a-select-option value="2">已结束</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="进场时间" >
              <a-date-picker placeholder="请输入进场时间" v-decorator="[ 'entryTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="完成时间" >
              <a-date-picker placeholder="请输入完成时间"  v-decorator="[ 'finishTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="要求部署时间" >
              <a-date-picker placeholder="请输入要求部署时间" v-decorator="['requireDeployTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程进度" >
              <a-textarea  placeholder="请输入工程进度" v-decorator="['progressOfItem', {}]" :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程地址" >
              <a-textarea  placeholder="请输入工程地址" v-decorator="['prjItemPlace', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   />
            </a-form-item>
          </a-col>
        </a-row>

      </a-form>
    </a-spin>
    <a-tabs type="card" v-show="isEdit">
    <a-tab-pane tab="添加设备" key="1" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
      <div style="float: right;">
        <a-button @click="showTender" type="primary" icon="plus">新增</a-button>
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
                  <a @click="handleDelete(record.outId)">删除</a>
                </span>
          </a-table>
        </div>
    </a-tab-pane>
    </a-tabs>
    <purchase-info-stock-show ref="PurchaseInfoStockShow" @ok="addProjectItem"></purchase-info-stock-show>

  </a-modal>
</template>

<script>
  import { httpAction,getAction,deleteAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import PurchaseInfoStockShow from './PurchaseInfoStockShow'
  import ACol from "ant-design-vue/es/grid/Col";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ARow from "ant-design-vue/es/grid/Row";
  import {filterObj} from '@/utils/util';

  export default {
    name: "projectItemModal",
    components: {PurchaseInfoStockShow, ACol, ATextarea, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        isEdit:false,
        isOk: true,
        projectId:"",
        companyId:"",
        thisMessage: "",
        dateFormat: 'YYYY-MM-DD',
        //字典数组缓存
        typeDictOptions: [],
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        //表头
        columns:[
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
            title: '设备名称',
            align: "center",
            dataIndex: 'equipName'
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipModel'
          },
          {
            title: '库存设备编号',
            align: "center",
            dataIndex: 'equipNo'
          },
          {
            title: '使用次数',
            align: "center",
            dataIndex: 'useTimes'
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
          total: 0,
        },
        loading: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/projectItem/add",
          edit: "/renche/projectItem/edit",
          checkCompany:"/renche/companyInfo/checkNameIsExsit",
          projectIdList:"/renche/projectItem/projectIdList",
          delOutId: "/renche/outEquip/delOutEquip",
        },
      }
    },
    created () {
      //初始化字典配置
      this.initDictConfig();
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 工程类型
        initDictOptions('PROJECTITEMTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
      },
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.projectIdList, params).then((res) => {
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
        param.projectId = this.projectId;
        return filterObj(param);
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
      add () {
        this.isEdit = false;
        this.edit({});
      },
      edit (record) {
        debugger;
        this.form.resetFields();
        this.dataSource = [];
        this.projectId = record.prjItemId;
        if(this.projectId != null && this.projectId != "",this.projectId != undefined){
          this.isEdit = true;
          this.loadData();
        }
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjItemName','prjItemNum','prjItemType','prjName','companyName','personInCharge','personTel','progressOfItem','prjItemPlace'))
          this.form.setFieldsValue({entryTime:this.model.entryTime?moment(this.model.entryTime):null});
          this.form.setFieldsValue({finishTime:this.model.finishTime?moment(this.model.finishTime):null});
          this.form.setFieldsValue({requireDeployTime:this.model.requireDeployTime?moment(this.model.requireDeployTime):null});
        });

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
              if(!this.model.prjItemId){
                httpurl+=this.url.add;
                method = 'post';
              }else{
                httpurl+=this.url.edit;
                method = 'put';
              }
              if(this.companyId != ''){
                this.model.belongCompany = this.companyId;
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
      handleDelete: function (id) {
        var that = this;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delOutId, {outId: id}).then((res) => {
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
      addProjectItem() {
        this.loadData();
      },
      showTender:function (){
        if(this.projectId==null || this.projectId == "" || this.projectId == undefined){
          this.$message.warning("请先保存基本信息，再添加设备");
          return;
        }
        this.$refs.PurchaseInfoStockShow.show(this.projectId);
        this.$refs.PurchaseInfoStockShow.title = "选择关联设备信息";
      },


      handleCancel () {
        this.close()
      },
      checkThisCompanyIsExsit(){
        // 触发表单验证
        this.form.validateFields(['companyName'], (errors, values) => {
          var companyName = values.companyName;//查询条件
          if(companyName != null && companyName != "") {
            getAction(this.url.checkCompany, {name: companyName}).then((res) => {
              if (res.success) {
                this.isOk = true;
                this.companyId = res.message;
              } else {
                this.isOk = false;
                if (res.code = '200') {
                  this.$message.warning(res.message);
                }
              }
            })
          }
        });
      }

    }
  }
</script>

<style scoped>

</style>