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
          <a-input placeholder="请输入设备名称"  v-decorator="['purchaseItem', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备型号"
          hasFeedback >
          <a-input placeholder="请输入设备型号" v-decorator="['itemModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
        </a-form-item>
        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="单价"
        hasFeedback >
        <a-input placeholder="请输入单价" v-decorator="['price', {rules: [{ required: true, message: '请输入单价' }]}]" />
      </a-form-item>
        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="采购数量"
        hasFeedback >
        <a-input   placeholder="请输入采购数量" v-decorator="['quantity', {rules: [{ required: true, message: '请输入采购数量' }]}]" />
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="总价"
          hasFeedback >
          <a-input   placeholder="总价" v-decorator="['totalPrice', {rules: [{ required: true }]}]" />
        </a-form-item>

        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="采购人员"
        hasFeedback >
        <a-input   placeholder="采购人员" v-decorator="['purchaser', {rules: [{ required: true,message: '请输入采购人员' }]}]" />
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss" v-decorator="[ 'purchaseTime', {rules: [{ required: true,message: '请选择采购时间' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCols"
          :wrapperCol="wrapperCols"
          label="采购来源"
          hasFeedback >
          <a-input   placeholder="采购来源" v-decorator="['purchaser', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="到货日期"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss" v-decorator="[ 'arrivalTime', {rules: [{ required: true,message: '请选择到货日期' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback >
          <a-input   placeholder="备注" v-decorator="['purchaser', {rules: [{ required: false,message: '请输入采购人员' }]}]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  //import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default{
    name:"prochaseInfoMode",
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
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/test/jeecgDemo/add",
          edit: "/test/jeecgDemo/edit",
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
          this.form.setFieldsValue(pick(this.model,'purchaseItem','itemModel','price','quantity','purchaser','purchaseTime','purchaser','arrivalTime','purchaser'))
          //时间格式化
       this.form.setFieldsValue({punchTime:this.model.punchTime?moment(this.model.punchTime,'YYYY-MM-DD HH:mm:ss'):null})
          this.form.setFieldsValue({birthday:this.model.birthday?moment(this.model.birthday):null})
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
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.punchTime = formData.punchTime?formData.punchTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.birthday = formData.birthday?formData.birthday.format():null;

            console.log(formData)
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