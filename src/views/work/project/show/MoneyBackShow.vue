<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="回款日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker v-decorator="[ 'backTime', {}]" disabled />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="回款金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number v-decorator="['backMoney', {}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" disabled/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开户银行" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['bank', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="银行账户" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['bankNo', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="负责人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['backPerson', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="邮箱" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['email', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="关联合同" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['contractName', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea readonly v-decorator="['content', {}]" :autosize="{ minRows: 2, maxRows: 6 }"  class="read_tex"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <div>
                <div v-for="(item,index) in model.filelist" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
        </a-row>

      </a-form>
    </a-spin>

    <ContractInfoShow ref="contractInfoShow" @func="modalFormOk"></ContractInfoShow>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import {queryDepartCGTreeList, doMian} from '@/api/api'
  import ContractInfoShow from '../modules/ContractInfoShow'

  export default {
    name: "moneyBackModel",
    components: {ATextarea,ContractInfoShow},
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
          add: "/renche/moneyBack/add",
          edit: "/renche/moneyBack/edit",
          fileUpload: doMian + "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
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
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
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
  .read_tex{
    background-color: #e6e5e56b;
    color: #8b89898a;
  }
</style>