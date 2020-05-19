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
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目名称" >
              <a-textarea placeholder="请输入项目名称" v-decorator="['prjName', {rules: [{ required: true,message: '请输入项目名称' }]}]" :autosize="{ minRows: 1, maxRows: 2 }" maxlength="150"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="招标编号">
              <a-input placeholder="请输入招标编号" v-decorator="['tenderNo', {rules: [{ required: true,message: '请输入招标编号' }]}]" maxLength="30"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="投标单位" >
              <a-input placeholder="请输入投标单位" v-decorator="['tenderCompany', {rules: [{ required: true,message: '请输入投标单位' }]}]" maxlength="30"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="招标代理机构">
              <a-input placeholder="请输入招标代理机构" v-decorator="['agency', {}]" maxLength="30"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="采购人" >
              <a-input placeholder="请输入采购人" v-decorator="['purchasePerson', {}]" maxLength="30"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="报价" >
              <a-input-number placeholder="请输入报价"  v-decorator="['tenderOffer', {initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="保证金" >
              <a-input-number id="deposit" placeholder="请输入保证金" v-decorator="['deposit', {initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" @blur="getReturnMoney" style="width: 60%;"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="服务费" >
              <a-input-number id="serviceMoney" placeholder="请输入服务费" v-decorator="['serviceMoney', {initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" @blur="getReturnMoney" style="width: 60%;"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="保证金是否退回" >
              <a-select id="isBack" v-decorator="['isBack', {}]" placeholder="请选择保证金是否已退回" @change="changeCheck">
                <a-select-option value="1">是</a-select-option>
                <a-select-option value="2">否</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="服务缴费方式" >
              <a-select id="payWay" v-decorator="['payWay', {}]" placeholder="请选择服务费缴费方式" @change="payWayChange">
                <a-select-option value="1">自缴</a-select-option>
                <a-select-option value="2">保证金扣除</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="应退保证金" >
              <a-input placeholder="0" v-decorator="['recedeDeposit', {}]" disabled/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划完成时间" >
              <a-date-picker format="YYYY-MM-DD" v-decorator="[ 'planOutTime', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际完成时间" >
              <a-date-picker format="YYYY-MM-DD" v-decorator="[ 'realityOutTime', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="交保证金时间" >
              <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss" v-decorator="[ 'payTime', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;"  v-show="isShow">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="退保证金时间" >
              <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss" v-decorator="[ 'recedeTime', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
              <a-textarea placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxlength="1500"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="22" style="padding-left: 22px;">
            <a-form-item label="附件" :wrapperCol="filewrapperCol" :labelCol="filelabelCol">
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
  </a-modal>
</template>

<script>
  import {getAction, httpAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "addTenderInfo",
    data() {
      return {
        title: "操作",
        visible: false,
        isShow: false,
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        filelabelCol: {
          xs: { span: 24 },
          sm: { span: 3 },
        },
        filewrapperCol: {
          xs: { span: 24 },
          sm: { span: 21 },
        },
        uploadLoading: false,
        headers: {},
        avatar: "",
        fileList: [],
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add: "/renche/tender/addTender",
          edit: "/renche/tender/upTender",
          fileUpload: "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {
      add() {
        this.edit({});
      },
      edit(record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.dataSource = [];
        this.fileList = [];
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.isShow = false;
        if(record.tenderId != undefined){
          if(record.isBack == "1"){
            this.isShow = true;
          }
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model, 'prjName', 'tenderNo', 'tenderCompany', 'planOutTime', 'realityOutTime', 'tenderOffer', 'deposit', 'isBack', 'agency', 'purchasePerson', 'serviceMoney', 'payWay','recedeDeposit', 'payTime', 'recedeTime'))
            //时间格式化
            this.form.setFieldsValue({payTime: this.model.payTime ? moment(this.model.payTime, 'YYYY-MM-DD HH:mm:ss"') : null});
            this.form.setFieldsValue({recedeTime: this.model.recedeTime ? moment(this.model.recedeTime, 'YYYY-MM-DD HH:mm:ss') : null});
            this.form.setFieldsValue({planOutTime: this.model.planOutTime ? moment(this.model.planOutTime, 'YYYY-MM-DD') : null});
            this.form.setFieldsValue({realityOutTime: this.model.realityOutTime ? moment(this.model.realityOutTime, 'YYYY-MM-DD') : null});
          });
        }
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
            if (!this.model.tenderId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'put';
            }
            values.recedeDeposit = that.model.recedeDeposit;
            that.model.fileRelId = that.avatar;
            let formData = Object.assign(this.model, values);
            formData.payTime = formData.payTime ? formData.payTime.format('YYYY-MM-DD HH:mm:ss') : null;
            formData.recedeTime = formData.recedeTime ? formData.recedeTime.format('YYYY-MM-DD HH:mm:ss') : null;
            formData.planOutTime = formData.planOutTime ? formData.planOutTime.format('YYYY-MM-DD') : null;
            formData.realityOutTime = formData.realityOutTime ? formData.realityOutTime.format('YYYY-MM-DD') : null;

            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok',res.result);
              } else {
                alert(res.message);
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
      changeCheck: function (val){
        if(val == "1"){
          this.isShow = true;
        }else{
          this.isShow = false;
        }
      },
      payWayChange: function(val){
        let isBack = document.getElementById("isBack").value.toString();
        this.getReturnMoney(isBack,val);
      },
      getReturnMoney: function (isBackVal,payWayVal){
        let deposit = document.getElementById("deposit").value.toString();
        let isBack = isBackVal;//是否退回保证金
        let totalPrice = "";
        let serviceMoney = document.getElementById("serviceMoney").value.toString();
        let payWay = payWayVal;//缴费方式
        if(payWay == "2"){//保证金扣除
          totalPrice = parseFloat(deposit) - parseFloat(serviceMoney);
        }else{
          totalPrice = deposit;
        }
        this.form.setFieldsValue({recedeDeposit: totalPrice});
        this.model.recedeDeposit = totalPrice;
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const prjName = this.model.prjName;
        const copyFile = new File([data.file], `项目${prjName}招标_${nowDate}_${timeStamp}_${data.file.name}`)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
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
    }
  }
</script>

<style scoped>

</style>