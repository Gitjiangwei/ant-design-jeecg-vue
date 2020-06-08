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
      <a-form :form="form">
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
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="采购数量">
              <a-input-number id="quantity" v-decorator="['quantity', {rules: [{ required: true,message: '请输入采购数量' }],initialValue: '1'}]" :min="1" :max="99999999999" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="单价">
              <a-input-number id="price" v-decorator="['price', {rules: [{ required: true,message: '请输入单价' }]}]" :min="0" :max="99999999999" style="width: 60%;" @blur="makeTotalMoney" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="总价">
              <a-input disabled placeholder="0" id="totalPrice" v-decorator="['totalPrice', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="拥有方式">
              <a-select  v-decorator="['haveWay', {rules: [{ required: true, message: '请选择拥有方式', }]}]" placeholder="请选择拥有方式" @change="changeHaveWay">
                <a-select-option value="0">租赁</a-select-option>
                <a-select-option value="1">购买</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="采购日期">
              <a-date-picker v-decorator="[ 'purchaseTime', {rules: [{ required: true,message: '请选择采购日期' }]}]" @change="dataChange"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="采购人员">
              <a-input placeholder="请输入采购人员" maxlength="6"v-decorator="['purchaser', {rules: [{ required: true,message: '请输入采购人员' }]}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="采购来源">
              <a-input placeholder="请输入所属公司" v-decorator="['companyName', {}]" @click="showCompanyList"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="到货日期">
              <a-date-picker v-decorator="[ 'arrivalTime', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" v-show="isShow">
            <a-form-item :labelCol="labelCol"   :wrapperCol="wrapperCol"  label="租赁到期日期">
              <a-date-picker v-decorator="['expirationDate', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
              <a-textarea  placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   maxLength="500" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="22" style="padding-left: 22px;">
            <a-form-item label="附件" :wrapperCol="filewrapperCol" :labelCol="filelabelCol">
              <a-upload
                name="file"
                :multiple="true"
                :headers="headers"
                :file-list="fileList"
                :customRequest="uploadFileRequest"
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

    <ShowMaterialInfoList ref="showMaterialInfoList" @func="modalFormOk"></ShowMaterialInfoList>
    <CompanyInfoShow ref="companyInfoShow" @func="companyModalFormOk"></CompanyInfoShow>

  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import CompanyInfoShow from "../../customer/modules/CompanyInfoShow"
  import ShowMaterialInfoList from '../../material/modules/ShowMaterialInfoList'

  export default {
    name: "prochaseInfoMode",
    components: {ShowMaterialInfoList, CompanyInfoShow},
    data() {
      return {
        isShow: false,
        title: "操作",
        visible: false,
        companyId: '',
        materialId: '',
        formData: {},
        fileList: [],
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
        filelabelCol: {
          xs: { span: 24 },
          sm: { span: 3 },
        },
        filewrapperCol: {
          xs: { span: 24 },
          sm: { span: 21 },
        },
        uploadLoading: false,
        headers: {},
        avatar: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add: "/renche/purchase/savePurchase",
          edit: "/renche/purchase/editPurchase",
          fileUpload: "/sys/common/upload",
        },
      }
    },
    created() {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
    },
    methods: {
      add() {
        this.edit({});
      },
      edit(record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'materialName','materialNo','materialType','materialUnit', 'price','totalPrice', 'quantity', 'purchaser', 'purchaseTime', 'companyName', 'remarks','haveWay'))
          //时间格式化
          this.form.setFieldsValue({purchaseTime: this.model.purchaseTime ? moment(this.model.purchaseTime) : null});
          this.form.setFieldsValue({arrivalTime: this.model.arrivalTime ? moment(this.model.arrivalTime) : null});
          this.form.setFieldsValue({expirationDate: this.model.expirationDate ? moment(this.model.expirationDate) : null});
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
            if (!this.model.purchaseId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'put';
            }

            that.model.fileRelId = that.avatar;
            let formData = Object.assign(this.model, values);
            formData.purchaseTime = formData.purchaseTime?formData.purchaseTime.format('YYYY-MM-DD'):null;
            formData.arrivalTime = formData.arrivalTime?formData.arrivalTime.format('YYYY-MM-DD'):null;
            formData.expirationDate = formData.expirationDate?formData.expirationDate.format('YYYY-MM-DD'):null;
            if(that.companyId != ''){
              formData.whichCompany = that.companyId;
            }
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
      makeTotalMoney(){
        let price = document.getElementById("price").value.toString();
        let quantity = document.getElementById("quantity").value.toString();
        if(price == ''){
          price = 0;
        }
        if(quantity == ''){
          quantity = 1;
        }
        var totalPrice = parseFloat(price) * parseInt(quantity);
        this.form.setFieldsValue({totalPrice: totalPrice})
        this.model.totalPrice = totalPrice;
      },
      changeHaveWay(val){
        if(val == '0'){
          this.isShow = true;
        }else{
          this.isShow = false;
        }
      },
      handleCancel() {
        this.close()
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
      showCompanyList:function (){
        this.$refs.companyInfoShow.show({});
        this.$refs.companyInfoShow.title = "选择客户信息";
      },
      companyModalFormOk(data){
        this.companyId = data.companyId;
        this.form.setFieldsValue({companyName:data.companyName});
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const materialName = this.model.materialName;
        const materialType = this.model.materialType;
        const copyFile = new File([data.file], `${materialType}${materialName}采购附件_${nowDate}_${timeStamp}_${data.file.name}`)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            data.onSuccess(res);
          }
        }).catch(({err}) => {
          /*f.onError()*/
        })
      },
      getDate() {
        let nowDate = new Date();
        let date = {
          year: nowDate.getFullYear(),
          month: nowDate.getMonth() + 1,
          date: nowDate.getDate(),
        }
        let systemDate = date.year + '-' + date.month + '-' +  date.date;
        return systemDate;
      },
      handleChange(info) {
        if(info.file.status == undefined){
          info.fileList.some((item,i) => {
            if(item.uid == info.file.uid){
              info.fileList.splice(i,1);
            }
          })
        }else{
          if (info.file.status === 'uploading') {
            this.uploadLoading = true
            this.fileList = [...info.fileList];
            return
          }
          if (info.file.status === 'done') {
            var response = info.file.response;
            this.uploadLoading = false;
            if (response.success) {
              this.avatar = response.message + "," + this.avatar;
              this.fileList = [...info.fileList];
              let fileItem = info.fileList.slice(-1);
              fileItem[0].id = response.message;
              Vue.set(info.fileList,info.fileList.length-1,fileItem[0])
            } else {
              this.$message.warning(response.message);
            }
            return
          }

          if(info.file.status === 'removed'){
            this.avatar = this.avatar.replace(info.file.id+',','')
          }
        }
      },
      dataChange(data, dataString){
        /*alert(data+"-----"+dataString);*/
      },
    }
  }

</script>
