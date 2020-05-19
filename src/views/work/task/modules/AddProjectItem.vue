<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
    @ok="handleOk"
    cancelText="关闭">

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="工程点信息" key="1" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程名称" >
                  <a-textarea placeholder="请输入工程名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入工程名称', }]}]" :autosize="{ minRows: 1, maxRows: 2 }" maxLength="150"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目名称" >
                  <a-textarea placeholder="请输入项目名称" v-decorator="['prjName', {rules: [{ required: true, message: '请输入工程名称', }]}]" :autosize="{ minRows: 1, maxRows: 2 }" maxLength="150"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程编号" >
                  <a-input placeholder="请输入工程编号" v-decorator="['prjItemNum', {}]" maxLength="30" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程类型" >
                  <a-select v-decorator="['prjItemType', {rules: [{ required: true, message: '请选择工程类型', }]}]" placeholder="请选择工程类型">
                    <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <!--<a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
                  <a-auto-complete placeholder="请输入所属公司" :open="isOpen"  @blur="getCompanyList" @select="chooseThis" v-decorator="['companyName', {rules: [{ required: true, message: '请输入正确公司名称', }]}]" maxLength="30">
                    <template slot="dataSource">
                      <a-select-option v-for="item in companyNameList" :key="item.companyId" :value="item.companyName">{{ item.companyName }}</a-select-option>
                    </template>
                  </a-auto-complete>
                </a-form-item>-->
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
                  <a-input placeholder="请输入所属公司" v-decorator="['companyName', {}]" @click="showCompanyList"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="负责人" >
                  <a-input placeholder="请输入负责人" v-decorator="['personInCharge', {}]"  maxLength="30"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="联系电话" >
                  <a-input placeholder="请输入联系电话" v-decorator="['personTel', {rules: [{ required: false,pattern: /^(0[0-9]{2,3}\-)?([2-9][0-9]{6,7})+(\-[0-9]{1,4})?$|^[1][3-9]\d{9}$/,message: '请输入正确联系电话' }]}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程状态" >
                  <a-select v-decorator="['prjItemStatus', {rules: [{ required: true, message: '请选择工程状态', }]}]" placeholder="请选择工程状态">
                    <a-select-option value="0">未开始</a-select-option>
                    <a-select-option value="1">进行中</a-select-option>
                    <a-select-option value="2">已结束</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="进场时间" >
                  <a-date-picker v-decorator="[ 'entryTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="完成时间" >
                  <a-date-picker v-decorator="[ 'finishTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="要求部署时间" >
                  <a-date-picker v-decorator="['requireDeployTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程进度" >
                  <a-textarea  placeholder="请输入工程进度" v-decorator="['progressOfItem', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="250" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程地址" >
                  <a-textarea  placeholder="请输入工程地址" v-decorator="['prjItemPlace', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   maxLength="250" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
                  <a-textarea  placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   maxLength="1500" />
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
      </a-tab-pane>
    </a-tabs>
    <CompanyInfoShow ref="companyInfoShow" @func="modalFormOk"></CompanyInfoShow>
  </a-modal>

</template>

<script>
  import { httpAction,getAction,deleteAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ACol from "ant-design-vue/es/grid/Col";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ARow from "ant-design-vue/es/grid/Row";
  import {filterObj} from '@/utils/util';
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import CompanyInfoShow from "./CompanyInfoShow";


  export default {
    name: "projectItemModal",
    components: {CompanyInfoShow, ACol, ATextarea, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        isUpload: false,
        formData: {},
        model: {},
        isOpen: false,
        projectId:"",
        isShow: false,
        companyId:"",
        contractId: "",
        contractName: "",
        chooseCompanyName: "",
        defaultActiveKey: "1",
        dateFormat: 'YYYY-MM-DD',
        uploadLoading: false,
        headers: {},
        avatar: "",
        fileList: [],
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
            title: '#',
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
            dataIndex: 'equipName'
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipModel'
          },
          {
            title: '库存设备编号',
            align: "center",
            dataIndex: 'equipNo'
          },
          {
            title: '使用次数',
            align: "center",
            dataIndex: 'useTimes'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
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
        loading: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{},
        companyNameList: [],
        url: {
          add: "/renche/projectItem/add",
          edit: "/renche/projectItem/edit",
          checkCompany:"/renche/companyInfo/checkNameIsExsit",
          projectIdList:"/renche/projectItem/projectIdList",
          delOutId: "/renche/outEquip/delOutEquip",
          searchCompany: "/renche/companyInfo/searchCompany",
          fileUpload: "/sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
      //初始化字典配置
      this.initDictConfig();
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 工程类型
        initDictOptions('PROJECTITEMTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
      },
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.projectIdList, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
            this.ipagination.total = res.result.total;
          }
        })
      },
      getQueryParams() {
        var param = {};
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        param.projectId = this.projectId;
        return filterObj(param);
      },
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadData();
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.form.resetFields();
        this.dataSource = [];
        this.fileList = [];
        this.projectId = record.prjItemId;
        this.companyId = record.belongCompany;
        if(this.projectId != undefined && this.projectId != null && this.projectId != ""){
          this.isShow = true;
          this.contractId = record.contractId;
          this.contractName = record.contractName;
          this.loadData();
        }
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjItemName','prjItemNum','prjItemType','prjName','prjItemStatus','companyName','personInCharge','personTel','progressOfItem','prjItemPlace'))
          this.form.setFieldsValue({entryTime:this.model.entryTime?moment(this.model.entryTime):null});
          this.form.setFieldsValue({finishTime:this.model.finishTime?moment(this.model.finishTime):null});
          this.form.setFieldsValue({requireDeployTime:this.model.requireDeployTime?moment(this.model.requireDeployTime):null});
        });

      },
      close () {
        this.$emit('close');
        this.isShow = false;
        this.isOpen = false;
        this.visible = false;
      },
      handleOk () {
        const that = this;
        if(that.companyId == ""){
          this.form.setFieldsValue({companyName:""});
        }
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.prjItemId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            if(this.companyId != ''){
              this.model.belongCompany = this.companyId;
            }
            that.model.fileRelId = that.avatar;
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
      callback: function(key){
        if(key != 1){
          if(this.model.prjItemId != undefined && this.model.prjItemId != ""){
            this.defaultActiveKey = key;
          }else{
            this.$message.warning("请先创建工程点信息");
          }
        }else{
          this.defaultActiveKey = key;
        }
      },
      handleCancel () {
        this.defaultActiveKey = "1";
        this.close()
      },
      getCompanyList(val){
        if(val != this.chooseCompanyName) {
          this.chooseCompanyName = '';
          getAction(this.url.searchCompany, {pageNo: "1",pageSize: "30",name: val}).then((res) => {
            if (res.success) {
              this.companyNameList = res.result.list;
              this.isOpen = true;
              if(this.companyNameList.length == 1){
                if(val == this.companyNameList[0].companyName){
                  this.companyId = this.companyNameList[0].companyId;
                }
              }else if(this.companyNameList.length == 0){
                this.companyId = "";
              }
            }
          })
        }
      },
      chooseThis: function(val,option){
        this.companyId = option.data.key;
        this.isOpen = false;
        this.chooseCompanyName = val;
      },
      beforeUpload: function (file) {
        return true;
        //TODO 验证文件大小
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const prjItemName = this.model.prjItemName
        const copyFile = new File([data.file], `工程点${prjItemName}_${nowDate}_${timeStamp}_${data.file.name}`)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            data.onSuccess(res);
          } else {
            this.$message.warning(res.message);
          }
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
      showCompanyList:function (){
        this.$refs.companyInfoShow.show();
        this.$refs.companyInfoShow.title = "选择客户信息";
      },
      modalFormOk(data) {
        this.companyId = data.companyId;
        this.form.setFieldsValue({companyName:data.companyName});
      },
    }
  }
</script>

<style scoped>

</style>