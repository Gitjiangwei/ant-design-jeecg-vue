<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目名称" >
              <a-textarea readonly v-decorator="['prjName', {}]" :autosize="{ minRows: 1, maxRows: 2 }" class="read_tex"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="招标编号">
              <a-input disabled v-decorator="['tenderNo', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="投标单位" >
              <a-input disabled v-decorator="['tenderCompany', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="招标代理机构">
              <a-input disabled v-decorator="['agency', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="采购人" >
              <a-input disabled v-decorator="['purchasePerson', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="报价" >
              <a-input disabled v-decorator="['tenderOffer', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="保证金" >
              <a-input disabled v-decorator="['deposit', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row>

          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="服务费" >
              <a-input disabled v-decorator="['serviceMoney', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="保证金是否退回" >
              <a-select disabled v-decorator="['isBack', {}]">
                <a-select-option value="1">是</a-select-option>
                <a-select-option value="2">否</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="服务缴费方式" >
              <a-select disabled v-decorator="['payWay', {}]" >
                <a-select-option value="1">自缴</a-select-option>
                <a-select-option value="2">保证金扣除</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="应退保证金" >
              <a-input placeholder="0" v-decorator="['recedeDeposit', {}]" disabled/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划完成时间" >
              <a-date-picker disabled v-decorator="[ 'planOutTime', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际完成时间" >
              <a-date-picker disabled v-decorator="[ 'realityOutTime', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="交保证金时间" >
              <a-date-picker disabled v-decorator="[ 'payTime', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="退保证金时间" >
              <a-date-picker disabled v-decorator="[ 'recedeTime', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
              <a-textarea disabled v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
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
  import { httpAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "addTenderInfo",
    data() {
      return {
        title: "操作",
        visible: false,
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
          add: "/renche/tender/addTender",
          edit: "/renche/tender/upTender",

        },
      }
    },
    created() {
    },
    methods: {

      add() {
        this.edit({});

      },
      edit(record) {
        //this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;

        if(record.tenderId != undefined){
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model, 'prjName', 'tenderNo', 'tenderCompany', 'planOutTime', 'realityOutTime', 'tenderOffer', 'deposit', 'isBack', 'agency', 'purchasePerson', 'serviceMoney', 'payWay','recedeDeposit', 'payTime', 'recedeTime'))
            //时间格式化
            this.form.setFieldsValue({payTime: this.model.payTime ? moment(this.model.payTime, 'YYYY-MM-DD HH:mm:ss"') : null});
            this.form.setFieldsValue({recedeTime: this.model.recedeTime ? moment(this.model.recedeTime, 'YYYY-MM-DD HH:mm:ss') : null});
            this.form.setFieldsValue({planOutTime: this.model.planOutTime ? moment(this.model.planOutTime, 'YYYY-MM-DD') : null});
            this.form.setFieldsValue({realityOutTime: this.model.realityOutTime ? moment(this.model.realityOutTime, 'YYYY-MM-DD') : null});
          });
        }
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
            if (!this.model.tenderId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'put';
            }
            values.recedeDeposit = that.model.recedeDeposit;
            let formData = Object.assign(this.model, values);
            formData.payTime = formData.payTime ? formData.payTime.format('YYYY-MM-DD HH:mm:ss') : null;
            formData.recedeTime = formData.recedeTime ? formData.recedeTime.format('YYYY-MM-DD HH:mm:ss') : null;
            formData.planOutTime = formData.planOutTime ? formData.planOutTime.format('YYYY-MM-DD') : null;
            formData.realityOutTime = formData.realityOutTime ? formData.realityOutTime.format('YYYY-MM-DD') : null;

            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok');
              } else {
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
      handleCancel() {
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