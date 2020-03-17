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
          label="发票名称"
          hasFeedback >
          <a-input placeholder="请输入发票名称"  v-decorator="['invociName', {rules: [{ required: true,message: '请输入发票名称' }]}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="开票时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD"  v-decorator="[ 'invociTime', {rules: [{ required: true,message: '请选择开票时间' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="发票内容">
          <a-input placeholder="请输入发票内容" v-decorator="['content', {rules: [{ required: true,message: '请输入发票内容' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="税号"
          hasFeedback >
          <a-input placeholder="请输入税号" v-decorator="['shuihao', {rules: [{ required: true,message: '请输入税号' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="单位地址"
          hasFeedback >
          <a-input placeholder="请输入单位地址" v-decorator="['address', {rules: [{ required: true,message: '请输入单位地址' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="电话号码"
          hasFeedback >
          <a-input placeholder="请输入电话号码" v-decorator="['tel', {rules: [{ required: true,message: '请输入电话号码' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="开户银行"
          hasFeedback >
          <a-input placeholder="请输入开户银行" v-decorator="['bank', {rules: [{ required: true,message: '请输入开户银行' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="银行账户"
          hasFeedback >
          <a-input placeholder="请输入银行账户" v-decorator="['bankNo', {rules: [{ required: true,message: '请输入银行账户' }]}]" />
        </a-form-item>

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

      </a-form>

    </a-spin>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "addInvoicInfo",
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

        uploadLoading: false,
        headers: {},
        avatar: "",
        isArris: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/invoic/addInvoic",
          edit: "/renche/invoic/upInvoic",

        },
      }
    },
    created () {
    },

    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },

    methods: {
      beforeUpload: function (file) {
        var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
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


      add () {

        this.edit({});

      },
      edit (record) {
        this.avatar = record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'invociName','invociTime','content','shuihao','address','tel','bank','bankNo','fileRelNum'))
          //时间格式化
             this.form.setFieldsValue({invociTime:this.model.invociTime?moment(this.model.invociTime,'YYYY-MM-DD'):null});

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
            if(!this.model.invociId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            let a = this.avatar.charAt(this.avatar.length - 1);
            debugger;
            if(a == ",") {
              this.avatar = this.avatar.substring(0, this.avatar.length - 1);
            }
            this.model.fileRelId = this.avatar;


            let formData = Object.assign(this.model, values);
            //时间格式化

            formData.invociTime = formData.invociTime?formData.invociTime.format('YYYY-MM-DD'):null;


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