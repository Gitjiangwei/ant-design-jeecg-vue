<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备名称"
          hasFeedback
        >
          <a-input placeholder="请输入设备名称" maxlength="30" v-on:focus="calculation" v-on:blur="calculation"
                   v-decorator="['purchaseItem', {rules: [{ required: true, message: '请输入设备名称', }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="设备型号"
          hasFeedback>
          <a-input placeholder="请输入设备型号" maxlength="15" v-on:focus="calculation" v-on:blur="calculation"
                   v-decorator="['itemModel', {rules: [{ required: true, message: '请输入设备型号' }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="单价"
          hasFeedback>
          <a-input placeholder="请输入单价" id="price" onblur="value=value.replace(/[^\d|.]/g,'')" onkeyup="value=value.replace(/[^\d|.]/g,'')" maxlength="6" v-on:focus="calculation" v-on:blur="calculation"
                   v-decorator="['price', {rules: [{ required: true, message: '请输入单价' }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购数量"
          hasFeedback>
          <a-input placeholder="请输入采购数量" onblur="value=value.replace(/[^\d]/g,'')" onkeyup="value=value.replace(/[^\d]/g,'')" id="quantity" maxlength="5" v-on:focus="calculation" v-on:blur="calculation"
                   v-decorator="['quantity', {rules: [{ required: true, message: '请输入采购数量' }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="总价"
          hasFeedback>
          <a-input disabled placeholder="0" readonly="true" id="totalPrice"
                   v-decorator="['totalPrice', {rules: [{ required: false,message:'请计算设备总价'}]}]"/>
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购人员"
          hasFeedback>
          <a-input placeholder="采购人员" v-on:focus="calculation" maxlength="6" v-on:blur="calculation"
                   v-decorator="['purchaser', {rules: [{ required: true,message: '请输入采购人员' }]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购时间"
          hasFeedback>
          <a-date-picker showTime format="YYYY-MM-DD" v-on:focus="calculation" v-on:blur="calculation"
                         v-decorator="[ 'purchaseTime', {rules: [{ required: true,message: '请选择采购时间' }]}]"/>
        </a-form-item>
        <a-form-item
          v-show="localMenuType!=0"
          label="采购来源"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol">
          <a-tree-select
            style="width:100%"
            :dropdownStyle="{ maxHeight: '200px', overflow: 'auto' }"
            :treeData="treeData"
            v-model="model.whichCompany"
            placeholder="请选择采购来源"
            :disabled="disableSubmit">
          </a-tree-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="到货日期"
          v-show = "isTimeShow"
          hasFeedback>
          <a-date-picker showTime format="YYYY-MM-DD" v-on:focus="calculation" v-on:blur="calculation"
                         v-decorator="[ 'arrivalTime', {}]"/>
        </a-form-item>
        <a-form-item style="display: none" class="isarr"
                     :labelCol="labelCol"
                     :wrapperCol="wrapperCol"
                     label="是否收货"
                     hasFeedback>
          <a-input style="display: none" class="isarr" v-on:focus="calculation" v-on:blur="calculation"
                   v-decorator="['isarrival', {rules: [{ required: false,message: '请输入采购来源' }]}]"/>
        </a-form-item>

        <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol"><!--  :before-upload="beforeUpload" -->
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

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback>
          <a-input placeholder="备注" v-on:focus="calculation" maxlength="300" v-on:blur="calculation"
                   v-decorator="['remarks', {rules: [{ required: false,message: '请输入采购人员' }]}]"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>

</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import {queryDepartCGTreeList, doMian} from '@/api/api'
  //import $ from 'jquery'

  export default {
    name: "prochaseInfoMode",
    components: {},
    data() {
      return {
        isShowFile: false,
        title: "操作",
        visible: false,
        isUpload: false,
        formData: {},
        fileList: [],
        model: {},
        isTimeShow: false,

        isarrivals: "",
        treeData: [],
        treeValue: '0-0-4',
        disableSubmit: false,
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        uploadLoading: false,
        headers: {},
        avatar: "",
        isArris: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add: "/renche/purchase/savePurchase",
          edit: "/renche/purchase/editPurchase",
          fileUpload: "/sys/common/upload",
          listKey: "/renche/purchase/qryPurchaseId",
        },
      }
    },
    created() {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};

    },
    methods: {
      loadTree() {
        var that = this;
        queryDepartCGTreeList().then((res) => {
          if (res.success) {
            that.treeData = [];
            let treeList = res.result;
            for (let a = 0; a < treeList.length; a++) {
              let temp = treeList[a];
              temp.isLeaf = temp.leaf;
              that.treeData.push(temp);
            }
          }
        });
      },
      load: function(){


      },
      beforeUpload: function (file) {
        /*     var fileType = file.type;
             if (fileType.indexOf('image') < 0) {
               this.$message.warning('请上传图片');
               return false;
             }*/
        return true;
        //TODO 验证文件大小
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const copyFile = new File([data.file], `${nowDate}_${timeStamp}_${data.file.name}`)
        console.log(copyFile)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            data.onSuccess(res);
          }
        }).catch(({err}) => {
          f.onError()
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
            console.log(response);
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

      add() {
        this.edit({});
      },
      edit(record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.isArris = true;
        getAction(this.url.listKey, {purchaseId:record.purchaseId}).then((res) => {
          if (res.success) {
           this.isarrivals = res.message;
          }
        });
        if(record.purchaseId!=null && record.purchaseId != "" && record.purchaseId != undefined){
          this.isTimeShow = true;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.fileList = [];
        this.loadTree();
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'purchaseItem', 'itemModel', 'price', 'totalPrice', 'quantity', 'purchaser', 'purchaseTime', 'whichCompany', 'arrivalTime', 'remarks'))
          //时间格式化
          this.form.setFieldsValue({purchaseTime: this.model.purchaseTime ? moment(this.model.purchaseTime, 'YYYY-MM-DD') : null});
          this.form.setFieldsValue({arrivalTime: this.model.arrivalTime ? moment(this.model.arrivalTime, 'YYYY-MM-DD') : null});
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        const that = this;
        this.calculation();
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
            if(this.avatar!=null && this.avatar != "" && this.avatar !=undefined) {
              let a = this.avatar.charAt(this.avatar.length - 1);
              debugger;
              if (a == ",") {
                this.avatar = this.avatar.substring(0, this.avatar.length - 1);
              }
            }
            this.model.fileRelId = this.avatar;

            debugger;
            var reg = new RegExp("[\\u4E00-\\u9FFF]+","g");
            if(reg.test(this.model.whichCompany)){
              this.model.whichCompany = this.isarrivals;
            }
            console.log(this.model);
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.purchaseTime = formData.purchaseTime ? formData.purchaseTime.format('YYYY-MM-DD') : null;
            formData.arrivalTime = formData.arrivalTime ? formData.arrivalTime.format('YYYY-MM-DD') : null;

            console.log(formData)
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
      calculation: function () {
        var price = document.getElementById("price").value.toString();
        var quantity = document.getElementById("quantity").value.toString();
        var totalPrice = parseFloat(price) * parseInt(quantity);
        if (price != "" && quantity != "") {
          document.getElementById("totalPrice").value = totalPrice;
        }

      },
      handleCancel() {
        this.close()
      },

    }
  }

</script>
