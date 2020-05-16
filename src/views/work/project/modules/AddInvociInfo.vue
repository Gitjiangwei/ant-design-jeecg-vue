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
            <a-form-item label="发票名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票名称"  v-decorator="['invociName', {rules: [{ required: true,message: '请输入发票名称' }]}]" maxLength="150" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="发票代码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票代码" v-decorator="['invociCode',{}]" maxLength="20" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="发票号码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票号码" v-decorator="['invociNumber', {}]" maxLength="20"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number id="price" v-decorator="['price', {rules: [{ required: true,message: '请输入正确金额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="税额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number id="shuiMoney" v-decorator="['shuiMoney', {rules: [{ required: true,message: '请输入正确税额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="税率" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number
                :min="0"
                :max="100"
                :step="0.01"
                style="width: 60%;"
                :formatter="value => `${value}%`"
                :parser="value => value.replace('%', '')"
                v-decorator="['shuiPercent', {rules: [{ required: false,message: '请输入正确税率' }],initialValue: '0'}]"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="0"  v-decorator="['totalMoney', {}]" disabled />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
           <!-- <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
              <a-auto-complete placeholder="请输入所属公司" :optionLabelProp="optionVal" :open="isOpen"  @blur="getCompanyList" @select="chooseThis" v-decorator="['companyName', {rules: [{ required: true, message: '请输入正确公司名称', }]}]" maxLength="30">
                <template slot="dataSource">
                  <a-select-option key="close">
                    <a-icon @click="hideDownList()"  type="close" style="float: right;"></a-icon>
                  </a-select-option>
                  <a-select-option v-for="item in companyNameList" :key="item.companyId" :value="item.companyName">{{ item.companyName }}</a-select-option>
                </template>
              </a-auto-complete>
            </a-form-item>-->
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
              <a-input placeholder="请输入所属公司" v-decorator="['companyName', {}]" @click="showCompanyList"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="公司税号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入公司税号" v-decorator="['shuihao', {rules: [{ required: true,pattern: /^\d*[a-z]*\d*[A-Z]*[a-z]*\d*[a-z]*[A-Z]*\d*[a-z]*[A-Z]*\d*[A-Z]*[a-z]*\d*[a-z]*$/,message: '请输入正确税号' }]}]" maxLength="20" />
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
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司地址" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入公司地址" v-decorator="['address', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="250"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="联系电话" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入联系电话" v-decorator="['tel', {rules: [{ required: false,pattern: /^(0[0-9]{2,3}\-)?([2-9][0-9]{6,7})+(\-[0-9]{1,4})?$|^[1][3-9]\d{9}$/,message: '请输入正确联系电话' }]}]" maxlength="20" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker v-decorator="[ 'invociTime', {rules: [{ required: true,message: '请选择开票日期' }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="发票内容" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入发票内容" v-decorator="['content', {rules: [{ required: true,message: '请输入发票内容' }]}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="255"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="签收人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入签收人" v-decorator="['signatory', {}]" maxLength="15" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="签收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker  v-decorator="[ 'signForTime', {}]" />
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

          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="签收单" :wrapperCol="wrapperCol" :labelCol="labelCol"><!--  :before-upload="beforeUpload" -->
              <a-upload
                name="file"
                :multiple="true"
                :headers="headers"
                :file-list="fileList1"
                :customRequest="uploadFileRequest1"
                @change="handleChange1"
              >
                <a-button>
                  <a-icon type="upload"/>
                  上传
                </a-button>
              </a-upload>
              <div>
                <div v-for="(item,index) in model.filelist1" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
        </a-row>

      </a-form>
    </a-spin>

    <ContractInfoShow ref="contractInfoShow" @func="modalFormOk"></ContractInfoShow>
    <CompanyInfoShow ref="companyInfoShow" @func="modalFormOk1"></CompanyInfoShow>
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
  import CompanyInfoShow from "../show/CompanyInfoShow";


  export default {
    name: "addInvociInfo",
    components: {CompanyInfoShow, ATextarea,ContractInfoShow},
    data () {
      return {
        title:"操作",
        visible: false,
        isUpload: false,
        formData: {},
        fileList:[],
        fileList1:[],
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
        avatar1: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        companyId:"",
        companyNameList: [],
        isOpen: false,
        chooseCompanyName: '',
        optionVal: '',
        textVal: '',
        url: {
          add: "/renche/invoci/add",
          edit: "/renche/invoci/edit",
          fileUpload: "/sys/common/upload",
          searchCompany: "/renche/companyInfo/searchCompany",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {
      beforeUpload: function (file) {
        /*     var fileType = file.type;
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
        const copyFile = new File([data.file], `${contracName}_${nowDate}_${"发票附件"}_${str1}_${timeStamp}_${str2}`)
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
      uploadFileRequest1(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const contracName = this.model.contractName;
        var fileName=data.file.name.toString();
        var str1=fileName.substring(0,fileName.lastIndexOf("."));
        var str2=fileName.substring(fileName.lastIndexOf("."), fileName.length);
        const copyFile = new File([data.file], `${contracName}_${nowDate}_${"签收单附件"}_${str1}_${timeStamp}_${str2}`)
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

      handleChange1(info) {
        if(info.file.status == undefined){
          info.fileList.some((item,i) => {
            if(item.uid == info.file.uid){
              info.fileList.splice(i,1);
            }
          })
        }else{
          if (info.file.status === 'uploading') {
            this.uploadLoading = true
            this.fileList1 = [...info.fileList];
            return
          }
          if (info.file.status === 'done') {
            var response = info.file.response;
            this.uploadLoading = false;
            console.log(response);
            if (response.success) {
              this.avatar1 = response.message + "," + this.avatar1;
              this.fileList1 = [...info.fileList];
              let fileItem = info.fileList.slice(-1);
              fileItem[0].id = response.message;
              Vue.set(info.fileList,info.fileList.length-1,fileItem[0])
            } else {
              this.$message.warning(response.message);
            }
            return
          }

          if(info.file.status === 'removed'){
            this.avatar1 = this.avatar1.replace(info.file.id+',','')
          }
        }
      },


      add () {
        this.edit({});
      },
      edit (record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.avatar1 = record.fileRelId1 == undefined?'':record.fileRelId1;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.fileList = [];
        this.fileList1 = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'invociName','invociCode','invociNumber','price','shuiMoney','shuiPercent','totalMoney','companyName','shuihao','bank','bankNo','address','content','signatory','contractName'))
          //时间格式化
          this.form.setFieldsValue({invociTime:this.model.invociTime?moment(this.model.invociTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({signForTime:this.model.signForTime?moment(this.model.signForTime,'YYYY-MM-DD'):null});
        });

      },
      close () {
        this.$emit('close')
        this.isOpen = false;
        this.chooseCompanyName = '';
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

            that.model.fileRelId = that.avatar;

            this.model.fileRelId1 = that.avatar1;
            values.totalMoney = that.model.totalMoney;
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
      makeTotalMoney(e){
        let price = document.getElementById("price").value.toString();
        let shuiMoney = document.getElementById("shuiMoney").value.toString();
        if(price == ""){
          price = "0";
        }
        if(shuiMoney == ""){
          shuiMoney = "0";
        }
        var totalPrice = parseFloat(price) + parseFloat(shuiMoney);
        if (price != "" || shuiMoney != "") {
          this.form.setFieldsValue({totalMoney: totalPrice})
          this.model.totalMoney = totalPrice;
        }
      },
      showContract:function (){
        this.$refs.contractInfoShow.show();
        this.$refs.contractInfoShow.title = "选择关联合同信息";
      },
      modalFormOk(data) {
        this.model.contractId = data[0].contractId;
        this.form.setFieldsValue({contractName:data[0].contractName});
      },
      modalFormOk1(data) {
        this.companyId = data.companyId;
        this.form.setFieldsValue({companyName:data.companyName});
      },

      getCompanyList(val){
        if(val != undefined && val != this.chooseCompanyName){
          this.textVal = val;
          this.companyId = '';
          this.chooseCompanyName = '';
          getAction(this.url.searchCompany, {pageNo: "1",pageSize: "30",name: val}).then((res) => {
            if (res.success) {
              this.companyNameList = res.result.list;
              this.isOpen = true;
              if(this.companyNameList.length == 1){
                if(val == this.companyNameList[0].companyName){
                  this.companyId = this.companyNameList[0].companyId;
                }
              }else if(this.companyNameList.length == 0){
                this.companyId = "";
              }else{
                this.isOpen = true;
              }
            }
          })
        }
      },
      chooseThis: function(val,option){
        this.isOpen = false;
        if(val != 'close'){
          this.companyId = option.key;
          this.optionVal = val;
          this.chooseCompanyName = val;
          this.companyNameList = [];
        }else{
          this.optionVal = this.textVal;
        }
      },
      hideDownList(){
        this.isOpen = false;
      },
      showCompanyList:function (){
        this.$refs.companyInfoShow.show();
        this.$refs.companyInfoShow.title = "选择客户信息";
      },

    }
  }
</script>

<style scoped>

</style>