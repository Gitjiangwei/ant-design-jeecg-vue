<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="数据字典编码"
          hasFeedback
        >
          <a-input placeholder="请输入数据字典编码"  maxlength="30" :disabled="isEdit"
                   v-decorator="['dictCodeId', {rules: [{ required: true, message: '请输入数据字典编码', },{ validator: this.validateDictCode}]}]"/>

        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="数据字典名称"
          hasFeedback
        >
          <a-input placeholder="请输入数据字典名称" maxlength="30"
                   v-decorator="['dictCodeName', {rules: [{ required: true, message: '请输入数据字典名称', }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="顺序"
          hasFeedback
        >
          <a-input placeholder="请输入顺序编号" maxlength="3"
                   v-decorator="['orderNo', {rules: [{ validator: this.validateDictOrderNo}]}]"/>
        </a-form-item>
      </a-form>
    </a-spin>

  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import {doMian,checkOnlyDictDetail} from '@/api/api'

  export default {
    name:"dictDetailMoudel",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        title: "操作",
        visible: false,
        dictItemId:"",
        model: {},
        disableSubmit: false,
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        uploadLoading: false,
        headers: {},
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
            add:"renche/dict/saveDict",
            edit:"renche/dict/updateDict",
        },
      }
    },
    created() {
      this.isEdit = true;
    },
    methods:{
      add(dictItemId) {
        this.dictItemId = dictItemId;
        this.isEdit = false;
        this.edit({});
      },
      edit(record,dictItemId) {
        if(record.dictId != null && record.dictId != "" && record.dictId != undefined){
          this.isEdit = true;
        }
        if(this.dictItemId ==""){
            this.dictItemId = dictItemId;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'dictCodeId', 'dictCodeName','orderNo'))
        });

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
            if (!this.model.dictId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
            formData.dictItemId = this.dictItemId;
            if(!this.model.dictId) {
              this.$confirm({
                title: "确认新增",
                content: "确认后数据字典编码将无法修改！",
                onOk: function () {
                  httpAction(httpurl, formData, method).then((res) => {
                    if (res.success) {
                      that.$message.success(res.message);
                      that.$emit('ok');
                    } else {
                      that.$message.warning(res.message);
                    }
                  }).finally(() => {
                    that.confirmLoading = false;
                    that.close();
                  })
                }
              });
            }else {
              httpAction(httpurl, formData, method).then((res) => {
                if (res.success) {
                  that.$message.success(res.message);
                  that.$emit('ok');
                } else {
                  that.$message.warning(res.message);
                }
              }).finally(() => {
                that.confirmLoading = false;
                that.close();
              })
            }


          }
        })
      },
      handleCancel() {
        this.close()
      },

      validateDictCode(rule, value, callback){
        var params = {
          dictCodeId:value,
          dictItemId:this.dictItemId
        };
        if(this.isEdit==true){
          callback();
        }
        if(/[\u4E00-\u95FA5]/g.test(value)){
          callback("数据字典编码不能输入中文，请重新输入");
        }else{
          checkOnlyDictDetail(params).then((res)=>{
            if(res.success){
              callback("该数据字典类型编码已存在，请重新输入");
            }else{
              callback();
            }
          });
        }
      },
      validateDictOrderNo(rule,value,callback){
        var reg = /^[0-9]*$/;
        if(!reg.test(value)){
          callback("顺序只能输入数字");
        }else {
          callback();
        }
      }
    }
  }
</script>