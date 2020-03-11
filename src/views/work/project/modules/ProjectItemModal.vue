<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工程名称"
          hasFeedback >
          <a-input placeholder="请输入工程名称" v-decorator="['prjItemName', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工程编号"
          hasFeedback >
          <a-input placeholder="请输入工程编号" v-decorator="['prjItemNum', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工程类型"
          hasFeedback >
          <a-select v-decorator="['prjItemType', {}]" placeholder="请选择工程类型">
            <a-select-option value="">请选择工程类型</a-select-option>
            <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="项目名称"
          hasFeedback >
          <a-input placeholder="请输入项目名称" v-decorator="['prjName', {}]" />
        </a-form-item>
        <a-form-item id="company"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="所属公司"
          >
          <a-input placeholder="请输入所属公司" v-decorator="['companyName', {}]" @blur="checkThisCompanyIsExsit"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="负责人"
          hasFeedback >
          <a-input placeholder="请输入负责人" v-decorator="['personInCharge', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="联系电话"
          hasFeedback >
          <a-input placeholder="请输入联系电话" v-decorator="['personTel', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="进场时间"
          hasFeedback >
          <a-date-picker v-decorator="[ 'entryTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工程进度"
          hasFeedback >
          <a-textarea  placeholder="请输入工程进度" v-decorator="['progressOfItem', {}]" :autosize="{ minRows: 2, maxRows: 6 }" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="完成时间"
          hasFeedback >
          <a-date-picker v-decorator="[ 'finishTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工程地址"
          hasFeedback >
          <a-textarea  placeholder="请输入工程地址" v-decorator="['prjItemPlace', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   />
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction,getAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "projectItemModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        isOk: true,
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

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/projectItem/add",
          edit: "/renche/projectItem/edit",
          checkCompany:"/renche/companyInfo/checkNameIsExsit",
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
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjItemName','prjItemNum','prjItemType','prjName','companyName','personInCharge','personTel','progressOfItem','prjItemPlace'))
          this.form.setFieldsValue({entryTime:this.model.entryTime?moment(this.model.entryTime):null});
          this.form.setFieldsValue({finishTime:this.model.finishTime?moment(this.model.finishTime):null});
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