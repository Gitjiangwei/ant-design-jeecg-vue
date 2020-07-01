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
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="关联工程点" >
              <a-input placeholder="请输入工程名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确工程点名称', }]}]" @click="showProjectItem" />
            </a-form-item>
          </a-col>

        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目类型" >
              <a-select v-decorator="['projectType', {}]" placeholder="请选择项目类型" @change="changeHaveWay">
                <a-select-option value="0">租赁</a-select-option>
                <a-select-option value="1">购买</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目内容" >
              <a-textarea  placeholder="请输入项目内容" v-decorator="['projectContent', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="完成情况" >
              <a-textarea  placeholder="请输入完成情况" v-decorator="['evaluate', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="承建单位意见" >
              <a-textarea  placeholder="请输入承建单位意见" v-decorator="['contractorOpinion', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row >
          <a-col :span="16" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="承建单位验收人">
              <a-input   placeholder="请输入承建单位验收人" v-decorator="['contractorPerson', {}]"   />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="签字日期">
              <a-date-picker    v-decorator="[ 'contractorSignatureDate', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="建设单位意见" >
              <a-textarea  placeholder="请输入建设单位意见" v-decorator="['buildOpinion', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row >
          <a-col :span="16" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="建设单位验收人">
              <a-input  placeholder="请输入建设单位验收人" v-decorator="['buildPerson', {}]"   />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="签字日期">
              <a-date-picker   v-decorator="[ 'buildSignatureDate', {}]" />
            </a-form-item>
          </a-col>
        </a-row>



        <a-row  v-show="isShow">
          <a-col :span="12" style="padding-left: 40px;"  >
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="租赁开始日期">
              <a-date-picker  v-decorator="['startDate1', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left:0px;"  >
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="租赁到期日期">
              <a-date-picker   v-decorator="['endDate1', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row>
          <a-col :span="22" style="padding-left: 22px;">
            <a-form-item :disabled="disabl" label="附件" :wrapperCol="filewrapperCol" :labelCol="filelabelCol">
              <a-upload
                name="file"
                :multiple="true"
                :headers="headers"
                :file-list="fileList"
                :customRequest="uploadFileRequest"
                @change="handleChange"
              >
                <a-button :disabled="disabl"  >
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
    <AddProjectItemRel ref="addProjectItemRel" @func="modalFormOk"></AddProjectItemRel>


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
  import AddProjectItemRel from "./AddProjectItemRel";



  export default {
    name: "ProjectReceiptInfoModel",
    components: {AddProjectItemRel, ATextarea},
    data () {
      return {
        title:"操作",
        visible: false,
        defaultActiveKey: '1',
        model: {},
        prjItemId: '',
        isShow:false,
        isUpload: false,
        formData: {},
        chargePersonList:[],
        prjItemNamesList:[],
        Id:"",
        fileList:[],
        fileList1:[],
        prjItemNames:[],
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
        isOpen: false,
        isOpen1: false,
        chooseProjectItemName: '',
        chooseChargePersonName:'',
        optionVal: '',
        optionVal1: '',
        textVal: '',
        url: {
          add: "/renche/projectReceipt/add",
          edit: "renche/projectReceipt/edit",
          getn:"/renche/workOrder/getn",
          //fileUpload: doMian + "/sys/common/upload",
          fileUpload: "/sys/common/upload",
          searchSysUser:"/renche/workOrder/userList",
          worSerlist: "/renche/WorkSerivice/qryWorkSerivice",
        },
      }
    },
    created () {
      //初始化工程点名稱列表
      this.initprojectNames();
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
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
        const prjItemName = this.model.prjItemName;
        const copyFile = new File([data.file], `工程验收单${prjItemName}_${nowDate}_${timeStamp}_${data.file.name}`)
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


      //初始化工程点名稱列表
      initprojectNames(){
        var that=this;
        getAction(this.url.getn).then((res)=> {

          if (res.success) {

            this.prjItemNames = res.result;
          } else {
            that.$message.warning(res.message);
          }
        })

      },

      add () {

        this.edit({});

      },
      edit (record) {

        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.avatar = record.fileRelId1 == undefined?'':record.fileRelId1;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        if(record.filelist1 == undefined){
          record.filelist1 = [];
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.fileList = this.model.filelist;
        this.visible = true;
        this.isUpload = false;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjItemName','projectType','projectContent','evaluate','contractorOpinion','contractorPerson','buildOpinion','buildPerson'))

          //时间格式化
          //  this.form.setFieldsValue({visitTime:this.model.visitTime?moment(this.model.visitTime,'YYYY-MM-DD HH:mm:ss'):null});
          this.form.setFieldsValue({buildSignatureDate:this.model.buildSignatureDate?moment(this.model.buildSignatureDate,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({contractorSignatureDate:this.model.contractorSignatureDate?moment(this.model.contractorSignatureDate,'YYYY-MM-DD '):null});
          this.form.setFieldsValue({startDate1:this.model.startDate?moment(this.model.startDate1,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({endDate1:this.model.endDate?moment(this.model.endDate1,'YYYY-MM-DD'):null});




        });

      },
      close () {
        this.$emit('close');
        this.isOpen = false;
        this.visible = false;
      },
      handleOk () {
        const that = this;
        that.isOpen = false;

        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.projectReceiptId){

              httpurl+=that.url.add;
              method = 'post';
            }else{

              httpurl+=that.url.edit;
              method = 'put';
            }
            that.model.fileRelId = that.avatar;
            let formData = Object.assign(that.model, values);

            //时间格式化
            formData.buildSignatureDate = formData.buildSignatureDate?formData.buildSignatureDate.format('YYYY-MM-DD'):null;
            formData.contractorSignatureDate = formData.contractorSignatureDate?formData.contractorSignatureDate.format('YYYY-MM-DD'):null;
            formData.startDate1 = formData.startDate1?formData.startDate1.format('YYYY-MM-DD'):null;
            formData.endDate1 = formData.endDate1?formData.endDate1.format('YYYY-MM-DD'):null;


            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
               that.$message.success(res.message);
                /*that.$emit('ok');*/
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
      handleCancel () {
        this.$emit('ok');
        this.defaultActiveKey = "1";
        this.close()
      },
      changeHaveWay(val){
        if(val == '0'){
          this.isShow = true;
        }else{
          this.isShow = false;
        }
      },
      hideDownList(){
        this.isOpen = false;
      },
      hideDownList1(){
        this.isOpen1 = false;
      },
      modalFormOk(data) {
        this.prjItemId = data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
      },
      showProjectItem:function (){
        this.$refs.addProjectItemRel.show();
        this.$refs.addProjectItemRel.title = "选择添加关联工程点信息";
      },

/*      addProjectItem(data) {
        this.prjItemId=data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
      },*/
    }
  }
</script>

<style scoped>
  .read_tex{
    background-color: #e6e5e56b;
    color: #8b89898a;
  }
</style>