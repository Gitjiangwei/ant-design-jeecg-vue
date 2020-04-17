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
          label="项目名称"
          hasFeedback >
          <a-input placeholder="请输入项目名称"  v-decorator="['prjName', {rules: [{ required: true,message: '请输入项目名称' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="招标编号">
          <a-input placeholder="请输入招标编号" v-decorator="['tenderNo', {rules: [{ required: true,pattern: /^\d*[a-z]*\d*[A-Z]*[a-z]*\d*[a-z]*$/,message: '请输入正确的招标编号' }]}]" maxLength="20"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="投标单位"
          hasFeedback >
          <a-input placeholder="请输入投标单位" v-decorator="['tenderCompany', {rules: [{ required: true,message: '请输入投标单位' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="报价"
          hasFeedback >
          <a-input-number :defaultValue="0" placeholder="请输入报价(单位：万元)" v-decorator="['tenderOffer', {rules: [{ required: true,message: '请输入报价' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;"  />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保证金"
          hasFeedback >
          <a-input-number :defaultValue="0" placeholder="请输入保证金(单位：万元)" v-decorator="['deposit', {rules: [{ required: true,message: '请输入保证金' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;"  />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保证金是否退回"
          hasFeedback >
          <a-select v-decorator="['isBack', {rules: [{ required: true,message: '请输入客户名称' }]}]" placeholder="请选择保证金是否已退回">
            <a-select-option :key="1">是</a-select-option>
            <a-select-option :key="2">否</a-select-option>
          </a-select>
        </a-form-item>
      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="招标代理机构"
        hasFeedback >
        <a-input placeholder="请输入招标代理机构" v-decorator="['agency', {rules: [{ required: true,message: '请输入招标代理机构' }]}]" />
      </a-form-item>
      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="采购人"
        hasFeedback >
        <a-input placeholder="请输入采购人" v-decorator="['purchasePerson', {rules: [{ required: true,message: '请输入采购人' }]}]" />
      </a-form-item>
      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="服务费"
        hasFeedback >
        <a-input-number :defaultValue="0" placeholder="请输入服务费" v-decorator="['serviceMoney', {rules: [{ required: true,message: '请输入服务费' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;"  />
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务费缴费方式"
          hasFeedback >
          <a-select v-decorator="['payWay', {rules: [{ required: true,message: '请输入服务费缴费方式' }]}]" placeholder="请选择服务费缴费方式">
            <a-select-option :key="1">自缴</a-select-option>
            <a-select-option :key="2">保证金扣除</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="计划完成时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD"  v-decorator="[ 'planOutTime', {rules: [{ required: true,message: '请选择计划完成时间' }]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="实际完成时间"
          hasFeedback >
          <a-date-picker showTime format="YYYY-MM-DD"  v-decorator="[ 'realityOutTime', {rules: [{ required: true,message: '请选择实际完成时间' }]}]" />
        </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="交保证金时间"
        hasFeedback >
        <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss"  v-decorator="[ 'payTime', {rules: [{ required: true,message: '请选择交保证金时间' }]}]" />
      </a-form-item>
      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="退保证金时间"
        hasFeedback >
        <a-date-picker showTime format="YYYY-MM-DD HH:mm:ss"  v-decorator="[ 'recedeTime', {}]" />
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
    name: "addTenderInfo",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
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
          add: "/renche/tender/addTender",
          edit: "/renche/tender/upTender",

        },
      }
    },
    created () {
    },
    methods: {

      add () {
        this.edit({});

      },
      edit (record) {
        //this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjName','tenderNo','tenderCompany','planOutTime','realityOutTime','tenderOffer','deposit','isBack','agency','purchasePerson','serviceMoney','payWay','payTime','recedeTime'))
             //时间格式化
          this.form.setFieldsValue({payTime:this.model.payTime?moment(this.model.payTime,'YYYY-MM-DD HH:mm:ss"'):null});
          this.form.setFieldsValue({recedeTime:this.model.recedeTime?moment(this.model.recedeTime,'YYYY-MM-DD HH:mm:ss'):null});
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
        var isBack1=this.model.isBack;
        var recedeTime1=this.model.recedeTime;
        alert("isBack1="+isBack1);
        alert("recedeTime1="+recedeTime1);
        if(isBack1==2||isBack1=="否"){
          if(recedeTime1!=undefined||recedeTime1!=''){
            alert("保证金未退回，请勿选择退回保证金时间！");
            return;
          }
        }
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.tenderId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            formData.payTime = formData.payTime?formData.payTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.recedeTime = formData.recedeTime?formData.recedeTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.planOutTime = formData.planOutTime?formData.planOutTime.format('YYYY-MM-DD'):null;
            formData.realityOutTime = formData.realityOutTime?formData.realityOutTime.format('YYYY-MM-DD'):null;

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
               /* that.$message.warning(res.message);*/
                alert(res.message);
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