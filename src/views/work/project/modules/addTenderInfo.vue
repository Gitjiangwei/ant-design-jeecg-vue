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
          label="项目名称"
          hasFeedback >
          <a-input placeholder="请输入项目名称"  v-decorator="['prjName', {rules: [{ required: true,message: '请输入项目名称' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="招标编号">
          <a-input placeholder="请输入招标编号" v-decorator="['tenderNo', {rules: [{ required: true,message: '请输入招标编号' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="投标单位"
          hasFeedback >
          <a-input placeholder="请输入投标单位" v-decorator="['tenderCompany', {rules: [{ required: true,message: '请输入投标单位' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="报价"
          hasFeedback >
          <a-input placeholder="请输入报价(单位：万元)" v-decorator="['tenderOffer', {rules: [{ required: true,message: '请输入报价' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保证金"
          hasFeedback >
          <a-input placeholder="请输入保证金(单位：万元)" v-decorator="['deposit', {rules: [{ required: true,message: '请输入保证金' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保证金是否退回"
          hasFeedback >
          <a-select v-decorator="['isBack', {rules: [{ required: true,message: '请输入客户名称' }]}]" placeholder="请选择保证金是否已退回">
            <a-select-option :key="1">是</a-select-option>
            <a-select-option :key="2">否</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>

    </a-spin>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "addTenderInfo",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
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
          add: "/renche/tender/addTender",
          edit: "/renche/tender/upTender",

        },
      }
    },
    created () {
    },
    methods: {

      add () {
        this.edit({});

      },
      edit (record) {

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjName','tenderNo','tenderCompany','tenderOffer','deposit','isBack'))
          /*   //时间格式化
             this.form.setFieldsValue({visitTime:this.model.visitTime?moment(this.model.visitTime,'YYYY-MM-DD'):null});
   */
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
            if(!this.model.tenderId){
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


    }
  }
</script>

<style scoped>

</style>