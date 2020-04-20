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
          <a-input placeholder="请输入设备名称"  maxlength="30"  v-decorator="['equipmentName', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备型号"
          hasFeedback >
          <a-input placeholder="请输入设备型号" maxlength="30"  v-decorator="['equipmentModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购数量"
          hasFeedback >
          <a-input   placeholder="请输入采购数量" maxlength="6"  v-decorator="['equipmentNumber', {rules: [{ required: true,validator: this.validateDictOrderNo}]}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback >
          <a-textarea  placeholder="请输入备注" maxlength="1000" v-decorator="['remarks', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'

  export default {
    name:"demandModules",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        title: "操作",
        visible: false,
        dictItemId:"",
        model: {},
        disableSubmit: false,
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        uploadLoading: false,
        headers: {},
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add:"renche/demand/saveDemand",
          edit:"renche/demand/updateDemand",
        },
      }
    },
    created() {
    },
    methods:{
      add() {
        this.edit({});
      },
      edit(record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'equipmentName', 'equipmentModel','equipmentNumber','remarks'))
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if (!this.model.dictId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
            formData.isSend = "1";
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok');
              } else {
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
        })
      },
      handleCancel() {
        this.close()
      },


      validateDictOrderNo(rule,value,callback){
        if(value==null||value==""||value==undefined){
          callback("请输入采购数量");
        }
        var reg = /^[0-9]*$/;
        if(!reg.test(value)){
          callback("采购数量只能输入数字");
        }else {
          callback();
        }
      }

    }
  }
</script>