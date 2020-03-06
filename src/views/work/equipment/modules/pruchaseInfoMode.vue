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
          <a-input placeholder="请输入设备名称" v-on:focus="calculation" v-on:blur="calculation"  v-decorator="['purchaseItem', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备型号"
          hasFeedback >
          <a-input placeholder="请输入设备型号" v-on:focus="calculation" v-on:blur="calculation" v-decorator="['itemModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
        </a-form-item>
        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="单价"
        hasFeedback >
        <a-input placeholder="请输入单价" id="price" v-on:focus="calculation" v-on:blur="calculation"  v-decorator="['price', {rules: [{ required: true, message: '请输入单价' }]}]" />
      </a-form-item>
        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="采购数量"
        hasFeedback >
        <a-input   placeholder="请输入采购数量"  id="quantity" v-on:focus="calculation" v-on:blur="calculation" v-decorator="['quantity', {rules: [{ required: true, message: '请输入采购数量' }]}]" />
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="总价"
          hasFeedback >
          <a-input  disabled  placeholder="0" readonly="true" id="totalPrice"  v-decorator="['totalPrice', {rules: [{ required: false,message:'请计算设备总价'}]}]" />
        </a-form-item>

        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="采购人员"
        hasFeedback >
        <a-input   placeholder="采购人员" v-on:focus="calculation" v-on:blur="calculation" v-decorator="['purchaser', {rules: [{ required: true,message: '请输入采购人员' }]}]" />
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD" v-on:focus="calculation" v-on:blur="calculation" v-decorator="[ 'purchaseTime', {rules: [{ required: true,message: '请选择采购时间' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCols"
          :wrapperCol="wrapperCols"
          label="采购来源"
          hasFeedback >
          <a-input   placeholder="采购来源" v-on:focus="calculation" v-on:blur="calculation" v-decorator="['whichCompany', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="到货日期"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD" v-on:focus="calculation" v-on:blur="calculation" v-decorator="[ 'arrivalTime', {rules: [{ required: true,message: '请选择到货日期' }]}]" />
        </a-form-item>
          <a-form-item style="display: none" class="isarr"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            label="是否收货"
            hasFeedback  >
            <a-input  style="display: none" class="isarr"  v-on:focus="calculation" v-on:blur="calculation" v-decorator="['isarrival', {rules: [{ required: false,message: '请输入采购来源' }]}]" />
          </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback >
          <a-input   placeholder="备注" v-on:focus="calculation" v-on:blur="calculation" v-decorator="['remarks', {rules: [{ required: false,message: '请输入采购人员' }]}]" />
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
        isArris:false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/purchase/savePurchase",
          edit: "/renche/purchase/editPurchase",
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
        this.isArris = true;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'purchaseItem','itemModel','price','totalPrice','quantity','purchaser','purchaseTime','whichCompany','arrivalTime','remarks'))
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
        this.calculation();
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.purchaseId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.purchaseTime = formData.purchaseTime?formData.purchaseTime.format('YYYY-MM-DD'):null;
            formData.arrivalTime = formData.arrivalTime?formData.arrivalTime.format('YYYY-MM-DD'):null;

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
      calculation: function(){
        var price = document.getElementById("price").value.toString();
        var quantity = document.getElementById("quantity").value.toString();
        var totalPrice = parseFloat(price)*parseInt(quantity);
        if(price!=""&&quantity!="") {
          document.getElementById("totalPrice").value = totalPrice;
        }

      },
      handleCancel () {
        this.close()
      },

    }
  }

</script>
