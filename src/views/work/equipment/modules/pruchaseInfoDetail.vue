<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleCancel"
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
          <a-input placeholder="请输入设备名称" disabled  v-decorator="['purchaseItem', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备型号"
          hasFeedback >
          <a-input placeholder="请输入设备型号" disabled  v-decorator="['itemModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="单价"
          hasFeedback >
          <a-input placeholder="请输入单价" disabled id="price"   v-decorator="['price', {rules: [{ required: true, message: '请输入单价' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购数量"
          hasFeedback >
          <a-input   disabled placeholder="请输入采购数量"  id="quantity"  v-decorator="['quantity', {rules: [{ required: true, message: '请输入采购数量' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="总价"
          hasFeedback >
          <a-input  disabled  placeholder="0" readonly="true" v-decorator="['totalPrice', {rules: [{ required: false,message:'请计算设备总价'}]}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购人员"
          hasFeedback >
          <a-input  disabled placeholder="采购人员"  v-decorator="['purchaser', {rules: [{ required: true,message: '请输入采购人员' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购时间"
          hasFeedback >
          <a-date-picker disabled showTime format="YYYY-MM-DD"  v-decorator="[ 'purchaseTime', {rules: [{ required: true,message: '请选择采购时间' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购来源"
          hasFeedback >
          <a-input disabled  placeholder="采购来源"  v-decorator="['whichCompany', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="到货日期"
          hasFeedback >
          <a-date-picker disabled showTime format="YYYY-MM-DD"  v-decorator="[ 'arrivalTime', {rules: [{ required: true,message: '请选择到货日期' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否收货"
          hasFeedback  >
          <a-input   disabled  v-decorator="['isarrival', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否入库"
          hasFeedback  >
          <a-input   disabled  v-decorator="['isstorage', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback >
          <a-input  disabled placeholder="备注"  v-decorator="['remarks', {rules: [{ required: false,message: '请输入采购人员' }]}]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import pick from 'lodash.pick'
  import moment from "moment"

  export default{
    name:"pruchaseInfoDetail",
    data(){
      return{
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
        isArris:false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
      }
    },
    created () {
    },
    methods: {
      detail (record){
        this.form.resetFields();
        if(record.isarrival==1){
          record.isarrival = "是";
        }else {
          record.isarrival = "否";
        }
        if(record.isstorage==1){
          record.isstorage = "已入库";
        }else {
          record.isstorage = "未入库";
        }
        this.model = Object.assign({},record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'purchaseItem','itemModel','price','quantity','totalPrice','purchaser','purchaseTime','whichCompany','arrivalTime','isarrival','isstorage','remarks'))
          //时间格式化
          this.form.setFieldsValue({purchaseTime:this.model.purchaseTime?moment(this.model.purchaseTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({arrivalTime:this.model.arrivalTime?moment(this.model.arrivalTime,'YYYY-MM-DD'):null});
        });
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },

    }
  }

</script>
