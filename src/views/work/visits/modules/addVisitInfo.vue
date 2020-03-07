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
          <a-select v-decorator="['companyName', {}]" placeholder="请选择客户名称">
            <a-select-option value="">请选择客户名称</a-select-option>
            <a-select-option v-for="item in companyNames" :key="item.value" :value="item.value">{{item.value}}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="拜访人"
          hasFeedback >
          <a-input placeholder="请输入拜访人"  v-decorator="['visitor', {}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="拜访时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD"  v-decorator="[ 'visitTime', {rules: [{ required: true,message: '请选择拜访时间' }]}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="拜访方式">
          <a-input placeholder="请输入拜访方式" v-decorator="['way', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="拜访内容"
          hasFeedback >
          <a-input placeholder="请输入拜访内容" v-decorator="['content', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="拜访结果"
          hasFeedback >
          <a-input placeholder="请输入拜访结果" v-decorator="['result', {}]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "addVisit",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        companyNames:[],
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
          add: "/renche/visit/add",
          edit: "/renche/visit/up",
          getn:"/renche/visit/getn",
        },
      }
    },
    created () {
     //初始化公司名稱列表
      this.initcompanyNames();


    },
    methods: {

      //初始化公司名稱列表
      initcompanyNames(){

        getAction(this.url.getn).then((res)=> {

          if (res.success) {
            this.companyNames = res.result;
          } else {
            that.$message.warning(res.message);
            alert(" " + res.message);
          }
        })

      },
      add () {
        this.edit({});

      },
      edit (record) {

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'companyName','visitor','visitTime','way','content','result'))
          //时间格式化
          this.form.setFieldsValue({visitTime:this.model.visitTime?moment(this.model.visitTime,'YYYY-MM-DD'):null});

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
            if(!this.model.visitId){

              httpurl+=this.url.add;
              method = 'post';
            }else{

              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);

            //时间格式化

            formData.visitTime = formData.visitTime?formData.visitTime.format('YYYY-MM-DD'):null;


            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                alert(" "+res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
                alert(" "+res.message);
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