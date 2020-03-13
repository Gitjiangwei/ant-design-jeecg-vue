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
              <a-input placeholder="请输入甲方公司" v-decorator="['partyA', {rules: [{ required: true, message: '请输入甲方公司', }]}]"  @blur="checkThisCompanyIsExsit" />
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
              <a-input placeholder="请输入乙方公司" v-decorator="['partyB', {rules: [{ required: true, message: '请输入乙方公司', }]}]"  @blur="checkThisCompanyIsExsit" />
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
                        <a-input placeholder="请输入合同金额" v-decorator="['contractMoney', {rules:[{pattern: /^-?\d+\.?\d*$/, message: '请输入正确数据', trigger: 'blur'}]}]" />
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
            <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入备注" v-decorator="['remark', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction,getAction} from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ARow from "ant-design-vue/es/grid/Row";
  import ATextarea from "ant-design-vue/es/input/TextArea";

  export default {
    name: "projectItemModal",
    components: {ATextarea, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
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

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/projectItem/add",
          edit: "/renche/projectItem/edit",
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
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'contractName','partyA','contractNoA','partyB','contractNoB','contractType','contractMoney','requireDeployTime','contractStatus','signInTime','remindPeriod','remark'))
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
          }
        })
      },
      handleCancel () {
        this.close()
      },
      checkThisCompanyIsExsit(e){
        // 触发表单验证
        // this.form.validateFields(['partyA','partyB'], (errors, values) => {
          var companyName = e.target.value;//查询条件
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
        // });
      }

    }
  }
</script>

<style scoped>

</style>