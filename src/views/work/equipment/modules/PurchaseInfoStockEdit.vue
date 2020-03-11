<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form" >

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备名称"
          hasFeedback
        >
          <a-input placeholder="请输入设备名称" disabled   v-decorator="['equipName', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="厂家设备编号"
          hasFeedback >
          <a-input placeholder="厂家设备编号"  v-decorator="['manufacoryNo', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备情况"
          hasFeedback >
          <a-input placeholder="请输入设备情况" id="price"   v-decorator="['condition', {}]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  //import $ from 'jquery'

  export default{
    name:"PurchaseInfoStockEdit",
    data(){
      return{
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        labelCols: {
          xs: { span: 18 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        wrapperCols: {
          xs: { span: 18 },
          sm: { span: 10 },
        },
        isArris:false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          edit: "/renche/equip/equipKeyDetail",
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
        debugger;
        this.isArris = true;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'equipName','manufacoryNo','condition'))
          //时间格式化
          this.form.setFieldsValue({purchaseTime:this.model.purchaseTime?moment(this.model.purchaseTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({arrivalTime:this.model.arrivalTime?moment(this.model.arrivalTime,'YYYY-MM-DD'):null});
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
            if(this.model.equipId){
              httpurl+=this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.purchaseTime = formData.purchaseTime?formData.purchaseTime.format('YYYY-MM-DD'):null;
            formData.arrivalTime = formData.arrivalTime?formData.arrivalTime.format('YYYY-MM-DD'):null;
            formData.equipStatus = "";
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
