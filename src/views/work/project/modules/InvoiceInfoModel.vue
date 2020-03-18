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
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开票时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker placeholder="请输入开票时间" v-decorator="['invoiceTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入开票金额(万元)" v-decorator="['invoiceMoney', {rules:[{pattern: /^-?\d+\.?\d{0,2}$/, message: '请输入正确数据', trigger: 'blur'}]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="回款时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker placeholder="请输入回款时间" v-decorator="['returnTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="回款金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入回款金额(万元)" v-decorator="['returnMoney', {rules:[{pattern: /^-?\d+\.?\d{0,2}$/, message: '请输入正确数据', trigger: 'blur'}]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="18" style="padding-left: 0px;padding-right: 0px;">
            <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="附件"
              hasFeedback>
              <!--  -->
              <a-upload
                name="file"
                :multiple="true"
                :action="uploadAction"
                :headers="headers"
                @change="handleChange"
              >
                <a-button>
                  <a-icon type="upload"/>
                  上传
                </a-button>
              </a-upload>
              <div>
                <div v-for="(item,index) in model.filelist" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>

    </a-spin>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import { doMian} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "invoiceInfoModel",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        initialFile: [],
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        uploadLoading: false,
        headers: {},
        avatar: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/invoiceInfo/add",
          edit: "/renche/invoiceInfo/edit",
          fileUpload: doMian + "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {

      add () {
        this.edit({});

      },
      edit (record) {
        this.avatar = record.fileRelId;

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'invoiceMoney','returnMoney','remark'))

          this.form.setFieldsValue({invoiceTime:this.model.invoiceTime?moment(this.model.invoiceTime):null});
          this.form.setFieldsValue({returnTime:this.model.returnTime?moment(this.model.returnTime):null});
        });

        this.model.filelist = [];
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
            if(!this.model.invoiceId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            this.model.fileRelId = this.avatar;

            let formData = Object.assign(this.model, values);

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('func');
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
      handleChange(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return;
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatar = response.message + "," + this.avatar;
            console.log(this.avatar);
          } else {
            this.$message.warning(response.message);
          }
        }
      },

    }
  }
</script>

<style scoped>

</style>