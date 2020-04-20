<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入客户名称"  v-decorator="['companyName', {rules: [{ required: true,message: '请输入客户名称' }]}]" maxLength="30" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="公司类型" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-select v-decorator="['type', {}]" placeholder="请选择客户类型">
                <a-select-option value="">请选择客户类型</a-select-option>
                <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="税号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入税号" v-decorator="['shuihao', {rules: [{ required: true,pattern: /^\d*[a-z]*\d*[A-Z]*[a-z]*\d*[a-z]*[A-Z]*\d*[a-z]*[A-Z]*\d*[A-Z]*[a-z]*\d*[a-z]*$/,message: '请输入正确税号' }]}]" maxLength="20"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开户银行" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入开户银行" v-decorator="['bank', {rules: [{ required: true,message: '请输入开户银行' }]}]"  maxLength="20"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="银行账户" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入银行账户" v-decorator="['bankNo', {rules: [{ required: true,pattern: /^\d{16,19}$/,message: '请输入正确银行账户' }]}]"  maxLength="20" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="联系人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入联系人" v-decorator="['contacts', {}]"  maxLength="20"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="身份证号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入身份证号" v-decorator="['idCard', {rules: [{ required: false,pattern: /^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/,message: '请输入正确的身份证号' }]}]"  maxLength="18" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="联系电话" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入联系电话" v-decorator="['phone', {rules: [{ required: false,pattern: /^(0[0-9]{2,3}\-)?([2-9][0-9]{6,7})+(\-[0-9]{1,4})?$|^[1][3-9]\d{9}$/,message: '请输入正确联系电话' }]}]"  maxLength="20"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="邮箱" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入邮箱" v-decorator="['email', {rules: [{ required: false,pattern: /^[A-Za-z0-9]+([_\.][A-Za-z0-9]+)*@([A-Za-z0-9\-]+\.)+[A-Za-z]{2,6}$/,message: '请输入正确邮箱' }]}]"  maxLength="50" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司简介" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入公司简介" v-decorator="['introduc', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="1500"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司地址" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入公司地址" v-decorator="['address', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="250"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol"><!--  :before-upload="beforeUpload" -->
              <a-upload
                name="file"
                :multiple="true"
                :headers="headers"
                :customRequest="uploadFileRequest"
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
  import { httpAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import { doMian} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "companyInfoModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        //字典数组缓存
        typeDictOptions: [],
        fileList: [],
        isUpload: false,
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
        validatorRules:{},
        uploadLoading: false,
        headers: {},
        formData: {},
        avatar: "",
        url: {
          add: "/renche/companyInfo/add",
          edit: "/renche/companyInfo/edit",
          fileUpload: "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
      //初始化字典配置
      this.initDictConfig();
    },
    computed: {
      uploadAction: function (data) {
        console.log("data",data)
        return this.url.fileUpload;
      }
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 客戶類型
        initDictOptions('CUSTOMETYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.isUpload = false;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'companyName','type','shuihao','bank','bankNo','contacts','idCard','phone','email','hobby','address'))
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
            if(!this.model.companyId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            that.model.fileRelId = that.avatar;
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
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const copyFile = new File([data.file], `${nowDate}_${timeStamp}_${data.file.name}`)
        console.log(copyFile)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            data.onSuccess(res);
          }
        }).catch(({err}) => {
          f.onError()
        })
      },
      getDate() {
        let nowDate = new Date();
        let date = {
          year: nowDate.getFullYear(),
          month: nowDate.getMonth() + 1,
          date: nowDate.getDate(),
        }
        let systemDate = date.year + '-' + date.month + '-' +  date.date;
        return systemDate;
      },
      handleChange(info) {
        if(info.file.status == undefined){
          info.fileList.some((item,i) => {
            if(item.uid == info.file.uid){
              info.fileList.splice(i,1);
            }
          })
        }else{
          if (info.file.status === 'uploading') {
            this.uploadLoading = true
            this.fileList = [...info.fileList];
            return
          }
          if (info.file.status === 'done') {
            var response = info.file.response;
            this.uploadLoading = false;
            console.log(response);
            if (response.success) {
              this.avatar = response.message + "," + this.avatar;
              this.fileList = [...info.fileList];
              let fileItem = info.fileList.slice(-1);
              fileItem[0].id = response.message;
              Vue.set(info.fileList,info.fileList.length-1,fileItem[0])
            } else {
              this.$message.warning(response.message);
            }
            return
          }

          if(info.file.status === 'removed'){
            this.avatar = this.avatar.replace(info.file.id+',','')
          }
        }
      },
    }
  }
</script>

<style scoped>

</style>