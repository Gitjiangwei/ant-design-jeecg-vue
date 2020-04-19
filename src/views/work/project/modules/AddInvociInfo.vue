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
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="发票名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票名称"  v-decorator="['invociName', {rules: [{ required: true,message: '请输入发票名称' }]}]" maxLength="150" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="发票代码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票代码" v-decorator="['invociCode',{}]" maxLength="20" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="发票号码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入发票号码" v-decorator="['invociNumber', {}]" maxLength="20"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number id="price" v-decorator="['price', {rules: [{ required: true,message: '请输入正确金额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="税额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number id="shuiMoney" v-decorator="['shuiMoney', {rules: [{ required: true,message: '请输入正确税额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="税率" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input-number
                :min="0"
                :max="100"
                :step="0.01"
                style="width: 60%;"
                :formatter="value => `${value}%`"
                :parser="value => value.replace('%', '')"
                v-decorator="['shuiPercent', {rules: [{ required: false,message: '请输入正确税率' }],initialValue: '0'}]"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="0"  v-decorator="['totalMoney', {}]" disabled />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="公司名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入公司名称" v-decorator="['companyName', {}]" maxLength="30" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="公司税号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入公司税号" v-decorator="['shuihao', {rules: [{ required: true,pattern: /^\d*[a-z]*\d*[A-Z]*[a-z]*\d*[a-z]*$/,message: '请输入正确税号' }]}]" maxLength="20" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开户银行" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入开户银行" v-decorator="['bank', {rules: [{ required: true,message: '请输入开户银行' }]}]"  maxLength="20"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="银行账户" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入银行账户" v-decorator="['bankNo', {rules: [{ required: true,pattern: /^\d{16,19}$/,message: '请输入正确银行账户' }]}]"  maxLength="20" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司地址" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入公司地址" v-decorator="['address', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="250"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="联系电话" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入联系电话" v-decorator="['tel', {rules: [{ required: false,pattern: /^(0[0-9]{2,3}\-)?([2-9][0-9]{6,7})+(\-[0-9]{1,4})?$|^[1][3-9]\d{9}$/,message: '请输入正确联系电话' }]}]" maxlength="20" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker v-decorator="[ 'invociTime', {rules: [{ required: true,message: '请选择开票日期' }]}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="发票内容" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea placeholder="请输入发票内容" v-decorator="['content', {rules: [{ required: true,message: '请输入发票内容' }]}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="255"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="签收人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请输入签收人" v-decorator="['signatory', {}]" maxLength="15" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="签收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker  v-decorator="[ 'signForTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="关联合同" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input placeholder="请选择关联合同" v-decorator="['contractName', {}]" @click="showContract" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-upload
                name="file"
                :multiple="true"
                :action="uploadAction"
                :headers="headers"
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

    <ContractInfoShow ref="contractInfoShow" @func="modalFormOk"></ContractInfoShow>

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
  import ContractInfoShow from './ContractInfoShow'

  export default {
    name: "addInvociInfo",
    components: {ATextarea,ContractInfoShow},
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
        uploadLoading: false,
        headers: {},
        avatar: "",
        isArris: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/renche/invoci/add",
          edit: "/renche/invoci/edit",
          fileUpload: doMian + "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },

    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },

    methods: {
      beforeUpload: function (file) {
        var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatar = response.message + "," + this.avatar;
            console.log(this.avatar);
          } else {
            this.$message.warning(response.message);
          }
        }
      },


      add () {
        this.edit({});
      },
      edit (record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'invociName','invociCode','invociNumber','price','shuiMoney','shuiPercent','totalMoney','companyName','shuihao','bank','bankNo','address','content','signatory','contractName'))
          //时间格式化
          this.form.setFieldsValue({invociTime:this.model.invociTime?moment(this.model.invociTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({signForTime:this.model.signForTime?moment(this.model.signForTime,'YYYY-MM-DD'):null});
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
            if(!this.model.invociId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            that.model.fileRelId = that.avatar;
            values.totalMoney = that.model.totalMoney;
            let formData = Object.assign(that.model, values);

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
      makeTotalMoney(e){
        let price = document.getElementById("price").value.toString();
        let shuiMoney = document.getElementById("shuiMoney").value.toString();
        if(price == ""){
          price = "0";
        }
        if(shuiMoney == ""){
          shuiMoney = "0";
        }
        var totalPrice = parseFloat(price) + parseFloat(shuiMoney);
        if (price != "" || shuiMoney != "") {
          this.form.setFieldsValue({totalMoney: totalPrice})
          this.model.totalMoney = totalPrice;
        }
      },
      showContract:function (){
        this.$refs.contractInfoShow.show();
        this.$refs.contractInfoShow.title = "选择关联合同信息";
      },
      modalFormOk(data) {
        this.model.contractId = data[0].contractId;
        this.form.setFieldsValue({contractName:data[0].contractName});
      },

    }
  }
</script>

<style scoped>

</style>