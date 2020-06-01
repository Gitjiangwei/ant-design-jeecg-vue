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
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="物料名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料名称"  v-decorator="['materialName', {rules: [{ required: true,message: '请输入物料名称' }]}]" maxLength="30" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="物料型号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料型号" v-decorator="['materialType', {rules: [{ required: true,message: '请输入物料型号' }]}]"  maxLength="30" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="物料单位" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入物料单位" v-decorator="['materialUnit', {rules: [{ required: true,message: '请输入物料单位' }]}]"  maxLength="5"/>
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

  export default {
    name: "MaterialInfoModal",
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
        validatorRules:{},
        formData: {},
        url: {
          add: "/renche/materialInfo/add",
          edit: "/renche/materialInfo/edit",
          fileUpload: "/sys/common/upload",
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
          this.form.setFieldsValue(pick(this.model,'materialNo','materialName','materialType','materialUnit'))
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
    }
  }
</script>

<style scoped>

</style>