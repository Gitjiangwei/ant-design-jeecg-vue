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
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工单名称">
          <a-input placeholder="请输入工单名称"  v-decorator="['workName', {rules: [{ required: true,message: '请输入工单名称' }]}]" />
        </a-form-item>
          </a-col>
        </a-row>

        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="负责人">
              <a-input placeholder="请输入负责人" v-decorator="['chargePerson', {rules: [{ required: true,message: '请输入负责人' }]}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="预计完成时间">
              <a-date-picker  format="YYYY-MM-DD "  v-decorator="[ 'completeTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="工程点">
              <a-select v-decorator="['prjItemName', {}]" placeholder="请选择工程点">
                <a-select-option value="">请选择工程点</a-select-option>
                <a-select-option v-for="item in prjItemNames" :key="item.value" :value="item.value">{{item.value}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="状态">
              <a-select v-decorator="['status', {}]" placeholder="请选择状态">
                <a-select-option value="1">实施</a-select-option>
                <a-select-option value="2">维修</a-select-option>
                <a-select-option value="3">拜访</a-select-option>
                <a-select-option value="4">销售</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>


    <a-row :gutter="24">
      <a-col :span="16" style="padding-left: 8px;">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="任务描述">
          <a-textarea placeholder="请输入任务描述" v-decorator="['describe', {}]" :row="3" />
        </a-form-item>
        </a-col>
      </a-row>
    <a-row :gutter="24">
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
  import {queryDepartCGTreeList, doMian} from '@/api/api'


  export default {
    name: "addWorkOrderInfo",
    components: {ATextarea},
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        fileList:[],
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
        url: {
          add: "/renche/workOrder/addWorkOrderInfo",
          edit: "/renche/workOrder/up",
          getn:"/renche/workOrder/getn",
          fileUpload: doMian + "/sys/common/upload",
        },
      }
    },
    created () {
      //初始化工程点名稱列表
      this.initprojectNames();
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}


    },

    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      load: function(){
        this.isEdit = true;
      },
      beforeUpload: function (file) {
       /* var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }*/
        return true;
        //TODO 验证文件大小
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
            } else {
              this.$message.warning(response.message);
            }
          }
        }
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
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.fileList = this.model.filelist;
        this.visible = true;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'workName','createPerson','describe','chargePerson','content','prjItemName','completeTime','status'));
          //时间格式化
          this.form.setFieldsValue({completeTime:this.model.completeTime?moment(this.model.completeTime,'YYYY-MM-DD'):null});

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
            if(!this.model.workId){

              httpurl+=this.url.add;
              method = 'post';
            }else{

              httpurl+=this.url.edit;
              method = 'put';
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
            formData.completeTime = formData.completeTime?formData.completeTime.format('YYYY-MM-DD'):null;
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