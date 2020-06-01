<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form" >
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="物料名称">
              <a-input placeholder="请输入设备名称" v-decorator="['materialName', {rules: [{ required: true, message: '请输入物料名称', }]}]" @click="showMaterialList" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="物料编号">
              <a-input disabled v-decorator="['materialNo', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="物料型号">
              <a-input disabled v-decorator="['materialType', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="单位">
              <a-input disabled v-decorator="['materialUnit', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="拥有方式">
              <a-select  v-decorator="['haveWay', {rules: [{ required: true, message: '请选择拥有方式', }]}]" placeholder="请选择拥有方式" @change="changeHaveWay">
                <a-select-option value="0">租赁</a-select-option>
                <a-select-option value="1">购买</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="需求数量">
              <a-input-number  v-decorator="['needNumber', {rules: [{ required: true,message: '请输入正确需求数量' }],initialValue: '1'}]" :min="1" :max="999999" :step="1" style="width: 60%;" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;"  v-show="isShow">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="租赁到期日期">
              <a-date-picker v-decorator="['expirationDate', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol" label="备注">
              <a-textarea placeholder="请输入备注" maxlength="1000" v-decorator="['remarks', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>

    <ShowMaterialInfoList ref="showMaterialInfoList" @func="modalFormOk"></ShowMaterialInfoList>

  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ShowMaterialInfoList from '../../material/modules/ShowMaterialInfoList';

  export default {
    name:"demandModules",
    components:{ShowMaterialInfoList},
    data(){
      return{
        isShow: false,
        isEdit:"",
        title: "操作",
        visible: false,
        prjItemId: '',
        materialId: '',
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add:"renche/demand/saveDemand",
          edit:"renche/demand/updateDemand",
        },
      }
    },
    created() {
    },
    methods:{
      add() {
        this.edit({});
      },
      edit(record) {
        this.prjItemId = record.prjItemId;
        // if(this.prjItemId != undefined && this.prjItemId != null && this.prjItemId != ""){
        //   this.isEdit=false;
        // }else {
        //   this.isEdit=true;
        // }
        if(record.haveWay == '0'){
          this.isShow = true;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'materialName','materialNo','materialType','materialUnit','needNumber','remarks','haveWay'));
          this.form.setFieldsValue({expirationDate:this.model.expirationDate?moment(this.model.expirationDate,'YYYY-MM-DD'):null});
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
        this.isShow = false;
      },
      handleOk() {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if (!this.model.demandId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
            formData.expirationDate = formData.expirationDate?formData.expirationDate.format('YYYY-MM-DD'):null;
            if(that.materialId != ''){
              formData.materialId = that.materialId;
            }
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
        })
      },
      handleCancel() {
        this.$emit('ok');
        this.close()
      },
      changeHaveWay(val){
        if(val == '0'){
          this.isShow = true;
        }else{
          this.isShow = false;
        }
      },
      showMaterialList(){
        this.$refs.showMaterialInfoList.show();
        this.$refs.showMaterialInfoList.title = "关联物料";
      },
      modalFormOk(data){
        this.form.setFieldsValue({materialName:data.materialName});
        this.form.setFieldsValue({materialNo:data.materialNo});
        this.form.setFieldsValue({materialType:data.materialType});
        this.form.setFieldsValue({materialUnit:data.materialUnit});
        this.materialId = data.materialId;
      },
    }
  }
</script>