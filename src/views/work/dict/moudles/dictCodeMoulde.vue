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
          label="字典类型编码"
          hasFeedback
        >
          <a-input placeholder="请输入字典类型编码"  maxlength="30" :disabled="isEdit"
                   v-decorator="['dictItemCode', {rules: [{ required: true, message: '请输入字典类型编码', },{ validator: this.validateDictCode}]}]"/>

        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="字典类型名称"
          hasFeedback
        >
          <a-input placeholder="请输入字典类型名称" maxlength="30"
                   v-decorator="['dictItemName', {rules: [{ required: true, message: '请输入字典类型编码', }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback
        >
          <a-textarea  placeholder="请输入备注" maxlength="2000" v-decorator="['remarks', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
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
  import {doMian,checkOnlyDict} from '@/api/api'

  export default {
    name:"dectCodeMoulde",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        title: "操作",
        visible: false,
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
          add:"/renche/dictitem/saveDictItem",
          edit:"/renche/dictitem/updateDictItem",
        },
      }
    },
    created() {
      this.isEdit = true;
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}
    },
    methods:{
      add() {
        this.isEdit = false;
        this.edit({});
      },
      edit(record) {
        if(record.dictItemId != null && record.dictItemId != "" && record.dictItemId != undefined){
          this.isEdit = true;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'dictItemCode', 'dictItemName','remarks'))
        });

      },
      close() {
        debugger;
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
            if (!this.model.dictItemId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
            if(!this.model.dictItemId) {
              this.$confirm({
                title: "确认新增",
                content: "确认后数据字典类型编码将无法修改！",
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
          dictItemCode:value
        };
        if(this.isEdit==true){
          callback();
        }
        if(/[\u4E00-\u95FA5]/g.test(value)){
          callback("数据字典类型编码不能输入中文，请重新输入");
        }else{
          checkOnlyDict(params).then((res)=>{
            if(res.success){
              callback("该数据字典类型编码已存在，请重新输入");
            }else{
              callback();
            }
          });
        }


      },
    }
  }
</script>