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
            <a-form-item label="客户名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-auto-complete placeholder="请输入客户名称"  @search="getCompanyListA" @select="chooseThisA" v-decorator="['companyName', {rules: [{ required: true, message: '请输入正确的客户名称', }]}]" maxLength="30">
                <template slot="dataSource">
                  <a-select-option v-for="item in companyNameListA" :key="item.companyName">{{ item.companyName }}</a-select-option>
                </template>
              </a-auto-complete>
            </a-form-item>
          </a-col>
        </a-row>


       <!-- <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="客户名称"
          hasFeedback >
          <a-select v-decorator="['companyName', {rules: [{ required: true,message: '请输入客户名称' }]}]" placeholder="请选择客户名称">
            <a-select-option value="">请选择客户名称</a-select-option>
            <a-select-option v-for="item in companyNames" :key="item.value" :value="item.value">{{item.value}}</a-select-option>
          </a-select>
        </a-form-item>-->

        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划执行时间">
              <a-date-picker  format="YYYY-MM-DD"  v-decorator="[ 'planExecuTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="实际执行时间">
              <a-date-picker format="YYYY-MM-DD"  v-decorator="[ 'realityExecuTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划完成时间">
              <a-date-picker  format="YYYY-MM-DD"  v-decorator="[ 'planOutTime', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际完成时间">
              <a-date-picker format="YYYY-MM-DD"  v-decorator="[ 'realityOutTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划参与人数">
              <a-input-number :defaultValue="0" placeholder="请输入计划参与人数" v-decorator="['planPersonNum', {initialValue: '0'}]" :min="0" :max="100000"   />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际参与人数">
              <a-input-number :defaultValue="0" placeholder="请输入实际参与人数" v-decorator="['realityPersonNum', { initialValue: '0'}]" :min="0" :max="100000"  />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="任务内容" >
              <a-textarea  placeholder="请输入任务内容" v-decorator="['content', {rules: [{ required: true,message: '请输入任务内容' }]}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="执行情况" >
              <a-textarea  placeholder="请输入执行情况" v-decorator="['result', {rules: [{ required: true,message: '请输入执行情况' }]}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="客户评价" >
              <a-select v-decorator="['evaluate', {}]" placeholder="请选择客户评价">
                <a-select-option value="1">很满意</a-select-option>
                <a-select-option value="2">满意</a-select-option>
                <a-select-option value="3">一般</a-select-option>
                <a-select-option value="4">较差</a-select-option>
                <a-select-option value="5">很差</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
              <a-textarea  placeholder="请输入备注"  v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>


        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-upload
                name="file"
                :multiple="true"
                :action="uploadAction"
                :headers="headers"
                :before-upload="beforeUpload"
                :file-list="fileList"
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
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import { doMian} from '@/api/api'


  export default {
    name: "addVisit",
    components: {ATextarea},
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        fileList: [],
        companyNames:[],
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
        companyNameListA: [],
        companyIdA:"",
        isEdit:true,
        isArris: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/WorkSerivice/upWorkSerivice",
          edit: "renche/WorkSerivice/upWorkSerivice",
          getn:"/renche/visit/getn",
          fileUpload: doMian + "/sys/common/upload",
          searchCompany: "/renche/companyInfo/searchCompany",
        },
      }
    },
    created () {
     //初始化公司名稱列表
      this.initcompanyNames();
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}

    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {

  /*    load: function(){
        this.isEdit = true;
      },*/
      beforeUpload: function (file) {
   /*     var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }*/
   return true;
        //TODO 验证文件大小
      },
      handleChange(info) {
        alert("info.file.status ="+info.file.status );
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
            } else {
              this.$message.warning(response.message);
            }
          }
        }
      },

      //初始化公司名稱列表
      initcompanyNames(){
        var that=this;
        getAction(this.url.getn).then((res)=> {

          if (res.success) {
            this.companyNames = res.result;
          } else {
            that.$message.warning(res.message);
            alert(" " + res.message);
          }
        })

      },
      getCompanyListA: function(val){
        getAction(this.url.searchCompany, {pageNo: "1",pageSize: "10",name: val}).then((res) => {
          if (res.success) {
            this.companyNameListA = res.result.list;
            if(this.companyNameListA.length == 1){
              if(val == this.companyNameListA[0].companyName){
                this.companyIdA = this.companyNameListA[0].companyId;
              }
            }else{
              this.companyIdA = "";
            }
          }
        })
      },
      chooseThisA: function(val){
        this.companyIdA = val;

      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.isEdit=false;
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.companyIdA = record.companyId;
        this.visible = true;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'planExecuTime','companyName','realityExecuTime','planOutTime','realityOutTime','planPersonNum','realityPersoNum','content','result','evaluate','remark'))
          //时间格式化
        //  this.form.setFieldsValue({visitTime:this.model.visitTime?moment(this.model.visitTime,'YYYY-MM-DD HH:mm:ss'):null});
          this.form.setFieldsValue({planExecuTime:this.model.planExecuTime?moment(this.model.planExecuTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({realityExecuTime:this.model.realityExecuTime?moment(this.model.realityExecuTime,'YYYY-MM-DD '):null});
          this.form.setFieldsValue({planOutTime:this.model.planOutTime?moment(this.model.planOutTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({realityOutTime:this.model.realityOutTime?moment(this.model.realityOutTime,'YYYY-MM-DD'):null});

        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        if(that.companyIdA == ""){
          this.form.setFieldsValue({companyName:""});
        }
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.workServiceId){

              httpurl+=this.url.add;
              method = 'post';
            }else{

              httpurl+=this.url.edit;
              method = 'put';
            }
            if(that.companyIdA != ''){
              this.model.partyA = that.companyIdA;
            }

            if(this.avatar!=null && this.avatar != "" && this.avatar !=undefined) {
              let a = this.avatar.charAt(this.avatar.length - 1);
              if (a == ",") {
                this.avatar = this.avatar.substring(0, this.avatar.length - 1);
              }
            }

            this.model.fileRelId = this.avatar;



            let formData = Object.assign(this.model, values);

            //时间格式化

           // formData.visitTime = formData.visitTime?formData.visitTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.planExecuTime = formData.planExecuTime?formData.planExecuTime.format('YYYY-MM-DD '):null;
            formData.setFieldsValue = formData.setFieldsValue?formData.setFieldsValue.format('YYYY-MM-DD '):null;
            formData.planOutTime = formData.planOutTime?formData.planOutTime.format('YYYY-MM-DD '):null;
            formData.realityOutTime = formData.realityOutTime?formData.realityOutTime.format('YYYY-MM-DD '):null;


            httpAction(httpurl,formData,method).then((res)=>{
              var meth = method;
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