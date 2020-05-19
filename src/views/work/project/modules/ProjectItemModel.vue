<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
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
              <a-col :span="10" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程类型" >
                  <a-select v-decorator="['prjItemType', {rules: [{ required: true, message: '请选择工程类型', }]}]" placeholder="请选择工程类型">
                    <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <!--  <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
                  <a-auto-complete placeholder="请输入所属公司" :optionLabelProp="optionVal"  :open="isOpen"  @blur="getCompanyList" @select="chooseThis" v-decorator="['companyName', {rules: [{ required: true, message: '请输入正确公司名称', }]}]" maxLength="30">
                     <template slot="dataSource">
                       <a-select-option key="close">
                         <a-icon @click="hideDownList()"  type="close" style="float: right;"></a-icon>
                       </a-select-option>
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
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="建设方式" >
                  <a-select v-decorator="['buildType', {rules: [{ required: true, message: '请选择建设方式', }]}]" placeholder="请选择建设方式">
                    <a-select-option value="0">迁移</a-select-option>
                    <a-select-option value="1">新建</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="拥有方式" >
                  <a-select v-decorator="['haveWay', {rules: [{ required: true, message: '请选择拥有方式', }]}]" placeholder="请选择拥有方式">
                    <a-select-option value="0">租赁</a-select-option>
                    <a-select-option value="1">购买</a-select-option>
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
              <a-col :span="12" style="padding-left: 0px;" v-show="isShow">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="关联合同" >
                  <a @click="showContractInfo">{{contractName}}</a>
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
          <a-row style="text-align: center;">
            <a-col>
              <span style="overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="handleOk" icon="check">保存</a-button>
              </span>
            </a-col>
          </a-row>
        </a-spin>
      </a-tab-pane>
      <a-tab-pane tab="关联设备" key="2" style="padding-bottom: 14px">
      <!--  <div style="float: right;">
          <a-button @click="showTender" type="primary" icon="plus">新增</a-button>
        </div>-->
        <div style="padding-top: 42px;">
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="prjItemId"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :loading="loading"
            @change="handleTableChange" style="padding-top: 10px;">

                <span slot="action" slot-scope="text, record">
                  <a @click="handleDelete(record.outId)">删除</a>
                </span>
          </a-table>
        </div>
      </a-tab-pane>
    </a-tabs>

    <purchase-info-stock-show ref="PurchaseInfoStockShow" @ok="addProjectItem"></purchase-info-stock-show>

    <ContractInfoLook ref="contractInfoShow"></ContractInfoLook>
    <CompanyInfoShow ref="companyInfoShow" @func="modalFormOk"></CompanyInfoShow>


    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  import { httpAction,getAction,deleteAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import PurchaseInfoStockShow from './PurchaseInfoStockShow'
  import ACol from "ant-design-vue/es/grid/Col";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ARow from "ant-design-vue/es/grid/Row";
  import {filterObj} from '@/utils/util';
  import ContractInfoLook from "../show/ContractInfoLook";
  import CompanyInfoShow from "../show/CompanyInfoShow";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "projectItemModal",
    components: {CompanyInfoShow, ContractInfoLook, PurchaseInfoStockShow, ACol, ATextarea, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        formData: {},
        model: {},
        isOpen: false,
        projectId:"",
        isShow: false,
        companyId:"",
        contractId: "",
        contractName: "",
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
        chooseCompanyName: '',
        optionVal: '',
        textVal: '',
        url: {
          add: "/renche/projectItem/add",
          edit: "/renche/projectItem/edit",
          checkCompany:"/renche/companyInfo/checkNameIsExsit",
          projectIdList:"/renche/projectItem/projectIdList",
          delOutId: "/renche/outEquip/delOutEquip",
          searchCompany: "/renche/companyInfo/searchCompany",
          searchContract: "/renche/contractInfo/getContractById",
          filelist: "/renche/purchase/fileList",
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
          this.form.setFieldsValue(pick(this.model,'buildType','haveWay','prjItemName','prjItemNum','prjItemType','prjName','prjItemStatus','companyName','personInCharge','personTel','progressOfItem','prjItemPlace'))
          this.form.setFieldsValue({entryTime:this.model.entryTime?moment(this.model.entryTime):null});
          this.form.setFieldsValue({finishTime:this.model.finishTime?moment(this.model.finishTime):null});
          this.form.setFieldsValue({requireDeployTime:this.model.requireDeployTime?moment(this.model.requireDeployTime):null});
        });

      },
      modalFormOk(data) {
        this.companyId = data.companyId;
        this.form.setFieldsValue({companyName:data.companyName});
      },
      close () {
        this.$emit('close');
        this.isShow = false;
        this.isOpen = false;
        this.chooseCompanyName = '';
        this.visible = false;
      },
      handleOk () {
        const that = this;
        that.isOpen = false;

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
            if(that.companyId != ''){
              this.model.belongCompany = that.companyId;

            }
            that.model.fileRelId = that.avatar;
            let formData = Object.assign(this.model, values);

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                if(method = 'post'){
                  this.model.prjItemId = res.result.prjItemId;
                }
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
        })
      },
      handleDelete: function (id) {
        var that = this;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delOutId, {outId: id}).then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.loadData();
              } else {
                that.$message.warning(res.message);
              }
            });
          }
        })
      },
      addProjectItem() {
        this.loadData();
      },
      showTender:function (){
        if(this.projectId==null || this.projectId == "" || this.projectId == undefined){
          this.$message.warning("请先保存基本信息，再添加设备");
          return;
        }
        this.$refs.PurchaseInfoStockShow.show(this.projectId);
        this.$refs.PurchaseInfoStockShow.title = "选择关联设备信息";
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
        this.$emit('ok');
        this.defaultActiveKey = "1";
        this.close()
      },
      getCompanyList(val){
        if(val != undefined && val != this.chooseCompanyName){
          this.textVal = val;
          this.companyId = '';
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
              } else{
                this.isOpen = true;
              }
            }
          })
        }
      },
      chooseThis: function(val,option){
        this.isOpen = false;
        if(val != 'close'){
          this.companyId = option.key;
          this.optionVal = val;
          this.chooseCompanyName = val;
          this.companyNameList = [];
        }else{
          this.optionVal = this.textVal;
        }
      },
      async showContractInfo(){
        let {result} = await getAction(this.url.searchContract, {contractId: this.contractId});
        let record = result;

        if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
          record.filelist = result.list;
        }

        if(record.elecFileRel != null || record.elecFileRel != "" || record.elecFileRel != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.elecFileRel});
          record.elefilelist = result.list;
        }
        this.$refs.contractInfoShow.edit(record);
        this.$refs.contractInfoShow.title = "合同详情";
      },

      showCompanyList:function (){
        this.$refs.companyInfoShow.show({});
        this.$refs.companyInfoShow.title = "选择客户信息";
      },
      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const prjItemName = this.model.prjItemName;
        const copyFile = new File([data.file], `工程点${prjItemName}_${nowDate}_${timeStamp}_${data.file.name}`)
        this.formData=new FormData();
        this.formData.append("file",copyFile);
        this.formData.append("headers",this.headers);
        httpAction(this.url.fileUpload,this.formData,"post").then((res)=>{
          if (res.success) {
            // this.avatar = res.message + "," + this.avatar;
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
      hideDownList(){
        this.isOpen = false;
      },
    }
  }
</script>

<style scoped>

</style>