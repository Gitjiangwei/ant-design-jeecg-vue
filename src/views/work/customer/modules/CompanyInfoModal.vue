<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="客户名称"
          hasFeedback >
          <a-input placeholder="请输入客户名称" v-decorator="['companyName', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="客户类型"
          hasFeedback >
          <a-select v-decorator="['type', {}]" placeholder="请选择客户类型">
            <a-select-option value="">请选择客户类型</a-select-option>
            <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="税号"
          hasFeedback >
          <a-input placeholder="请输入税号" v-decorator="['shuihao', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="开户银行">
          <a-input placeholder="请输入开户银行" v-decorator="['bank', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="银行帐号"
          hasFeedback >
          <a-input placeholder="请输入银行帐号" v-decorator="['bankNo', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="联系人"
          hasFeedback >
          <a-input placeholder="请输入联系人" v-decorator="['contacts', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="身份证号"
          hasFeedback >
          <a-input placeholder="请输入身份证号" v-decorator="['idCard', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="联系电话"
          hasFeedback >
          <a-input placeholder="请输入联系电话" v-decorator="['phone', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="邮箱"
          hasFeedback >
          <a-input placeholder="请输入邮箱" v-decorator="['email', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="个人爱好"
          hasFeedback >
          <a-textarea  placeholder="请输入个人爱好" v-decorator="['hobby', {}]" :row="2" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="公司地址"
          hasFeedback >
          <a-textarea placeholder="请输入公司地址" v-decorator="['address', {}]" :row="2" />
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "companyInfoModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        //字典数组缓存
        typeDictOptions: [],
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
        validatorRules:{
        },
        url: {
          add: "/renche/companyInfo/add",
          edit: "/renche/companyInfo/edit",
        },
      }
    },
    created () {
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
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'companyName','type','shuihao','bank','bankNo','contacts','idCard','phone','email','hobby','address'))
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
            if(!this.model.companyId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);

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


    }
  }
</script>

<style scoped>

</style>