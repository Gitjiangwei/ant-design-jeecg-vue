<template>
  <a-modal
    :title="title"
    :width="900"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form" >
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="设备名称">
              <a-input placeholder="请输入设备名称"  maxlength="30"  v-decorator="['equipmentName', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="设备型号">
              <a-input placeholder="请输入设备型号" maxlength="30"  v-decorator="['equipmentModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"  :wrapperCol="wrapperCol"  label="拥有方式">
              <a-select v-decorator="['haveWay', {rules: [{ required: true, message: '请选择拥有方式', }]}]" placeholder="请选择拥有方式">
                <a-select-option value="0">租赁</a-select-option>
                <a-select-option value="1">购买</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol"  :wrapperCol="wrapperCol"  label="需要数量">
              <a-input-number v-decorator="['equipmentNumber', {rules: [{ required: true,message: '请输入正确需要数量' }],initialValue: '1'}]" :min="0" :max="999999" :step="1" style="width: 60%;" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注">
              <a-textarea  placeholder="请输入备注" maxlength="1000" v-decorator="['remarks', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
            </a-form-item>
          </a-col>
        </a-row>



      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'

  export default {
    name:"demandModel",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        title: "操作",
        visible: false,
        dictItemId:"",
        demandList: [],
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
          saveSession:"renche/demand/getSessionDemand",
        },
      }
    },
    created() {
    },
    methods:{
      add(record) {
        this.edit(record);
      },
      edit(record) {
        this.form.resetFields();
        this.demandList = record.demandList;
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'equipmentName', 'equipmentModel','equipmentNumber','remarks','haveWay'))
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
            this.model.makeDemand = '0';//需要设备，还没生成需求
            let httpurl = this.url.saveSession;
            let method = 'post';
            let formData = Object.assign(this.model, values);
            formData.demandList = this.demandList;

            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok',res.result);
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
    }
  }
</script>