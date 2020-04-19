<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="发票名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea readonly v-decorator="['invociName', {}]" :autosize="{ minRows: 1, maxRows: 2 }" class="read_tex" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="发票代码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['invociCode',{}]" disabled />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="发票号码" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['invociNumber', {}]" disabled/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开票金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['price', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="税额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['shuiMoney', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="税率" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['shuiPercent', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="总额" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['totalMoney', {}]" disabled />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="公司名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['companyName', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="公司税号" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['shuihao', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="开户银行" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['bank', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="银行账户" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input v-decorator="['bankNo', {}]"  disabled />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="公司地址" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea disabled v-decorator="['address', {}]" :autosize="{ minRows: 2, maxRows: 6 }" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="联系电话" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['tel', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="开票日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker disabled v-decorator="[ 'invociTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="发票内容" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-textarea readonly v-decorator="['content', {}]" :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="签收人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['signatory', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item label="签收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-date-picker disabled  v-decorator="[ 'signForTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item label="关联合同" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-input disabled v-decorator="['contractName', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <div>
                <div v-for="(item,index) in model.filelist" :key="index">{{item.fileName}}</div>
              </div>
            </a-form-item>
          </a-col>
        </a-row>

      </a-form>
    </a-spin>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  import pick from 'lodash.pick'
  import moment from "moment"
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "invociInfoShow",
    components: {ATextarea},
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
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {
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
      handleCancel () {
        this.close()
      },
    }
  }
</script>

<style scoped>
  .read_tex{
    background-color: #e6e5e56b;
    color: #8b89898a;
  }
</style>