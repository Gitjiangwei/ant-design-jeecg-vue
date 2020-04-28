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
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="回款日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker v-decorator="[ 'backTime', {rules: [{ required: true,message: '请选择回款日期' }]}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="回款金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number v-decorator="['backMoney', {rules: [{ required: true,message: '请输入正确回款金额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="回款开户银行" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入回款开户银行" v-decorator="['bank', {rules: [{ required: true,message: '请输入回款开户银行' }]}]"  maxLength="20"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="回款银行账户" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入回款银行账户" v-decorator="['bankNo', {rules: [{ required: true,pattern: /^\d{16,19}$/,message: '请输入回款银行账户' }]}]"  maxLength="20" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="负责人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入负责人" v-decorator="['backPerson', {}]" maxLength="15" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="邮箱" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入邮箱" v-decorator="['email', {rules: [{ required: false,pattern: /^[A-Za-z0-9]+([_\.][A-Za-z0-9]+)*@([A-Za-z0-9\-]+\.)+[A-Za-z]{2,6}$/,message: '请输入正确邮箱' }]}]" maxLength="50" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="关联合同" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请选择关联合同" v-decorator="['contractName', {}]" @click="showContract" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入备注" v-decorator="['content', {rules: [{ required: false,message: '请输入备注' }]}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="255"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol"><!--  :before-upload="beforeUpload" -->
              <a-upload
                name="file"
                :multiple="true"
                :headers="headers"
                :file-list="fileList"
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

    <ContractInfoShow ref="contractInfoShow" @func="modalFormOk"></ContractInfoShow>

  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import { doMian} from '@/api/api'
  import ContractInfoShow from './ContractInfoShow'

  export default {
    name: "moneyBackModel",
    components: {ATextarea,ContractInfoShow},
    data () {
      return {
        title:"操作",
        visible: false,
        isUpload: false,
        formData: {},
        model: {},
        fileList:[],
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
          add: "/renche/moneyBack/add",
          edit: "/renche/moneyBack/edit",
          fileUpload: "/sys/common/upload",

        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },

    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },

    methods: {
      beforeUpload: function (file) {
       /* var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }*/
       return true;
        //TODO 验证文件大小
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const contracName = this.model.contractName;
        var fileName=data.file.name.toString();
        var str1=fileName.substring(0,fileName.lastIndexOf("."));
        var str2=fileName.substring(fileName.lastIndexOf("."), fileName.length);
        const copyFile = new File([data.file], `${contracName}_${nowDate}_${str1}_${timeStamp}_${str2}`)
        console.log(copyFile)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            // this.avatar = res.message + "," + this.avatar;
            data.onSuccess(res);
          } else {
            this.$message.warning(res.message);
          }
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
      add () {
        this.edit({});
      },
      edit (record) {

        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'backMoney','bank','bankNo','backPerson','email','remark','contractName'))
          //时间格式化
          this.form.setFieldsValue({backTime:this.model.backTime?moment(this.model.backTime,'YYYY-MM-DD'):null});
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
            if(!this.model.backId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            that.model.fileRelId = that.avatar;
            let formData = Object.assign(that.model, values);

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
      showContract:function (){
        this.$refs.contractInfoShow.show();
        this.$refs.contractInfoShow.title = "选择关联合同信息";
      },
      modalFormOk(data) {
        this.model.contractId = data[0].contractId;
        this.form.setFieldsValue({contractName:data[0].contractName});
      },
      checkNumber(e){
        let value = e.target.value;
        if(!value || new RegExp(/^[1-9]?\d*\d(\.\d{1,2})?$/).test(value)){
          let zs = "0";
          let xs = "0";
          if(value != "" && value.indexOf(".") > -1){
            zs = value.substring(0,value.indexOf("."));
            xs = value.substring(value.indexOf(".")+1,value.length);
          }else{
            zs = value;
          }
          if(new RegExp(/^[0]*$/).test(zs)){
            zs = "0";
          }
          if(xs == "00" || xs == "0"){
            e.target.value = zs;
          }else{
            e.target.value = zs + "." + xs;
          }
        }
      }
    }
  }
</script>

<style scoped>

</style>