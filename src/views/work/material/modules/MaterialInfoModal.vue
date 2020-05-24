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
     <!--   <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="物料编号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料编号"  v-decorator="['materialNo', {rules: [{ required: true,message: '请输入物料编号' }]}]" maxLength="100" />
            </a-form-item>
          </a-col>
        </a-row>-->
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="物料名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料名称"  v-decorator="['materialName', {rules: [{ required: true,message: '请输入物料名称' }]}]" maxLength="30" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="名称拼音" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="名称拼音（大写）" v-decorator="['pyName', {rules: [{ required: true,message: '请输入名称拼音（大写）' }]}]"  maxLength="30"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="物料型号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料型号" v-decorator="['materialType', {rules: [{ required: true,message: '请输入物料型号' }]}]"  maxLength="20" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="物料单位" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料单位" v-decorator="['materialUnit', {rules: [{ required: true,message: '请输入物料单位' }]}]"  maxLength="20"/>
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
    name: "MaterialInfoModal",
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
          add: "/renche/materialInfo/add",
          edit: "/renche/materialInfo/edit",
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
          this.form.setFieldsValue(pick(this.model,'materialNo','materialName','pyName','materialType','materialUnit'))
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

            if(!this.model.materialId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            alert("httpurl="+httpurl)
            that.model.fileRelId = that.avatar;
            let formData = Object.assign(this.model, values);

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok',res.result);
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
        const companyName = this.model.companyName;
        const copyFile = new File([data.file], `公司${companyName}_${nowDate}_${timeStamp}_${data.file.name}`)
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