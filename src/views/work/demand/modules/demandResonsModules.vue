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
          label="退回原因"
          hasFeedback >
          <a-textarea  placeholder="请输入退回原因" :disabled="isEdit" maxlength="1000" v-decorator="['reasons', {rules: [{ required: true,message: '请输入退回原因'}]}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import {deleteAction} from "../../../../api/manage";


  export default {
    name:"demandResonsModules",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        title: "操作",
        visible: false,
        query:false,
        demandId:"",
        dis:false,
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
          edit:"renche/demand/updateTuihui",
        },
      }
    },
    created() {
    },
    methods: {
      add(demandId) {
        debugger;
        this.demandId = demandId;
        this.dis=true;
        this.edit({});
      },
      edit(record) {
        this.form.resetFields();
        if(!this.dis) {
          this.isEdit = true;
          this.query = true;
        }
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'reasons'))
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        if(this.query){
          this.confirmLoading = false;
          this.close();
          return;
        }
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = this.url.edit;
            let formData = Object.assign(this.model, values);
            deleteAction(httpurl, {demandId:that.demandId,reasons:formData.reasons}).then((res) => {
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
    }
  }
</script>