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
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="关联工程点" >
              <a-input placeholder="请输入工程点名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确工程点名称', }]}]" @click="showProjectItem" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
         <!--   <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="接收单位">
              <a-input placeholder="请输入接收单位" v-decorator="['receiver', {rules: [{ required: true,message: '请输入接收单位' }]}]" maxLength="30"/>
            </a-form-item>
-->
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="接收单位" >
              <a-input placeholder="请输入接收单位" v-decorator="['receiver', {rules: [{ required: true,message: '请输入接收单位' }]}]" @click="showCompanyList"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 40px;">
          <!--  <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="归还单位">
              <a-input placeholder="请输入归还单位" v-decorator="['returnCompany', {rules: [{ required: true,message: '请输入归还单位' }]}]" maxLength="30"/>
            </a-form-item>-->
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="归还单位" >
              <a-input placeholder="请输入归还单位" v-decorator="['returnCompany', {rules: [{ required: true,message: '请输入归还单位' }]}]" @click="showCompanyList1"/>
            </a-form-item>

          </a-col>

        </a-row>


        <a-row>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="开始日期" >
              <a-date-picker format="YYYY-MM-DD" v-decorator="[ 'startDate', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="结束日期" >
              <a-date-picker format="YYYY-MM-DD" v-decorator="[ 'endDate', {}]"/>
            </a-form-item>
          </a-col>

        </a-row>

        <a-row>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="接收人" >
              <a-input placeholder="请输入接收人" v-decorator="['recipient', {rules: [{ required: true,message: '请输入接收人' }]}]" maxlength="30"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="接收人电话">
              <a-input placeholder="请输入接收人电话"  v-decorator="['phoneNumber',validatorRules.phoneNumber]" maxLength="11"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 0px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="归还人" >
              <a-input placeholder="请输入归还人" v-decorator="['returnPerson', {rules: [{ required: true,message: '请输入归还人' }]}]" maxlength="30"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 40px;" >
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="归还人电话">
              <a-input placeholder="请输入归还人电话"  v-decorator="['returnPhoNum',validatorRules.phoneNumber]" maxLength="11"/>
            </a-form-item>
          </a-col>
        </a-row>


      </a-form>
    </a-spin>
    <div>
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="prjItemId"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading">
      <!--  @change="" style="padding-top: 10px;">-->
                <!--<span slot="action" slot-scope="text, record">
                  <a @click="handleDelete(record.outId)">删除</a>
                </span>-->
      </a-table>
    </div>
    <AddProjectItemRel ref="addProjectItemRel" @func="modalFormOk"></AddProjectItemRel>
    <CompanyInfoShow ref="companyInfoShow" @func="modalFormOk1"></CompanyInfoShow>
    <CompanyInfoShow ref="companyInfoShow1" @func="modalFormOk2"></CompanyInfoShow>
  </a-modal>

</template>


<script>
  import { httpAction , getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import AddProjectItemRel from "./AddProjectItemRel";
  import CompanyInfoShow from "../../customer/modules/CompanyInfoShow";

  export default {
    name: "LeaseReturnModel",
    components: {AddProjectItemRel,CompanyInfoShow},

    data() {
      return {
        title: "操作",
        visible: false,
        isShow: false,
        isCheck:false,
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        filelabelCol: {
          xs: { span: 24 },
          sm: { span: 3 },
        },
        filewrapperCol: {
          xs: { span: 24 },
          sm: { span: 21 },
        },

        //表头
        columns:[
          {
            title: '序号',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '设备名称',
            align: "center",
            dataIndex: 'materialName'
          },
          {
            title: '设备单位',
            align: "center",
            dataIndex: 'materialUnit'
          },
          {
            title: '设备数量',
            align: "center",
            dataIndex: 'needNumber'
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'remarks'
          },

        ],
        //数据集
        dataSource: [],
        // 分页参数
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0,
        },


        uploadLoading: false,
        headers: {},
        avatar: "",
        fileList: [],
        confirmLoading: false,
        form: this.$form.createForm(this),

        validatorRules:{

          phoneNumber:{rules: [{validator:this.validateMobile}]}
        },
        url: {
          edit: "/renche/leaseReturn/edit",
          add: "/renche/leaseReturn/addLeaseReturn",
          fileUpload: "/sys/common/upload",
          demand:"/renche/leaseReturn/demand",
          check:"/renche/leaseReturn/check",
        },

      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {
      add() {
        this.edit({});
      },
      edit(record) {
        this.dataSource = record.demandList;
        console.log(record);
        this.fileList = [];
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.isShow = false;
        if(record.leaseReturnId != undefined){
          if(record.isBack == "1"){
            this.isShow = true;
          }
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model, 'prjItemName', 'receiver', 'recipient', 'phoneNumber','demandList','startDate','endDate','returnCompany','returnPerson','returnPhoNum'))
            //时间格式化
            this.form.setFieldsValue({startDate: this.model.startDate ? moment(this.model.startDate, 'YYYY-MM-DD "') : null});
            this.form.setFieldsValue({endDate: this.model.endDate ? moment(this.model.endDate, 'YYYY-MM-DD "') : null});
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
            if (!this.model.leaseReturnId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            formData.endDate = formData.endDate ? formData.endDate.format('YYYY-MM-DD') : null;
            formData.startDate = formData.startDate ? formData.startDate.format('YYYY-MM-DD') : null;
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok',res.result);
              } else {
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
      changeCheck: function (val){
        if(val == "1"){
          this.isShow = true;
        }else{
          this.isShow = false;
        }
      },
      validateMobile(rule,value,callback) {
        if (!value || new RegExp(/^1([38][0-9]|4[579]|5[0-3,5-9]|6[6]|7[0135678]|9[89])\d{8}$/).test(value)) {
          callback();
        } else {
          callback("您的手机号码格式不正确!");
        }
      },
      showProjectItem:function (){
        this.$refs.addProjectItemRel.show();
        this.$refs.addProjectItemRel.title = "选择添加关联工程点信息";
      },
      modalFormOk(data) {
        this.prjItemId = data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
        var that=this;

        getAction(that.url.check,{prjItemId: this.prjItemId}).then((res) => {
          if (res.success) {
            /*that.$message.success(res.message);*/
            that.isCheck=true;
          } else {
            alert(res.message);
            that.isCheck=false;
            that.form.setFieldsValue({prjItemName:""});
          }
        })
        if(that.isCheck){
          getAction(that.url.demand,{prjItemId: this.prjItemId}).then((res) => {
            if (res.success) {
             /* that.$message.success(res.message);*/
              that.dataSource = res.result;
              console.log(res.result)
              console.log(that.dataSource)
            } else {
              alert(res.message);
            }
          })
        }
       /*.finally(() => {
          that.confirmLoading = false;
          that.close();
        })*/
      },

      showCompanyList:function (){
        this.$refs.companyInfoShow.show();
        this.$refs.companyInfoShow.title = "选择客户信息";
      },
      showCompanyList1:function (){
        this.$refs.companyInfoShow1.show();
        this.$refs.companyInfoShow1.title = "选择客户信息";
      },
      modalFormOk1(data) {
        this.model.companyId = data.companyId;

        this.form.setFieldsValue({receiver:data.companyName});
      },
      modalFormOk2(data) {
        this.model.companyId = data.companyId;

        this.form.setFieldsValue({returnCompany:data.companyName});
      },






     /* getDate() {
        let nowDate = new Date();
        let date = {
          year: nowDate.getFullYear(),
          month: nowDate.getMonth() + 1,
          date: nowDate.getDate(),
        }
        let systemDate = date.year + '-' + date.month + '-' +  date.date;
        return systemDate;
      },*/


    }
  }
</script>

<style scoped>

</style>
