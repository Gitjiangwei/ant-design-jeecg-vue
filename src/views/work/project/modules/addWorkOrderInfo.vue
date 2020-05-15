<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">


    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="工单基本信息" key="1" style="padding-bottom: 14px">
       <a-spin :spinning="confirmLoading">
        <a-form :form="form">

        <a-row :gutter="24">
          <a-col :span="16" style="padding-left: 8px;">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="工单名称">
          <a-input placeholder="请输入工单名称"  v-decorator="['workName', {rules: [{ required: true,message: '请输入工单名称' }]}]" />
        </a-form-item>
          </a-col>
        </a-row>
        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
         <!--   <a-form-item label="工程点" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-auto-complete placeholder="请输入工程点" :optionLabelProp="optionVal" :open="isOpen"  @blur="getPrjItemNamesList" @select="chooseThisB" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确的工程点名称', }]}]" maxLength="30">
                <template slot="dataSource">
                  <a-select-option key="close">
                    <a-icon @click="hideDownList()"  type="close" style="float: right;"></a-icon>
                  </a-select-option>
                  <a-select-option v-for="item in prjItemNamesList" :key="item.prjItemName" :value="item.prjItemName">{{ item.prjItemName }}</a-select-option>
                </template>
              </a-auto-complete>
            </a-form-item>-->
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="关联工程点" >
              <a-input placeholder="请输入工程点名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确工程点名称', }]}]" @click="showProjectItem" />
            </a-form-item>

          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="类型">
              <a-select v-decorator="['status', {}]" placeholder="请选择工单类型">
                <a-select-option value="1">实施</a-select-option>
                <a-select-option value="2">维修</a-select-option>
                <a-select-option value="3">拜访</a-select-option>
                <a-select-option value="4">销售</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

        </a-row>




        <a-row >
          <a-col :span="12" style="padding-left: 40px;">
            <a-form-item label="负责人" :wrapperCol="wrapperCol" :labelCol="labelCol">
              <a-auto-complete placeholder="请输入负责人"  :open="isOpen1"  @blur="getChargePersonList" @select="chooseThisA" v-decorator="['chargePerson', {rules: [{ required: true, message: '请输入正确的负责人名称', }]}]" maxLength="30">
                <template slot="dataSource">
                  <a-select-option v-for="item in chargePersonList" :key="item.prjItemId" :value="item.username">{{ item.username }}</a-select-option>
                </template>
              </a-auto-complete>
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 0px;" float:left>
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="预计完成时间">
              <a-date-picker  format="YYYY-MM-DD "  v-decorator="[ 'completeTime', {}]" />
            </a-form-item>
          </a-col>
        </a-row>


    <a-row :gutter="24">
      <a-col :span="16" style="padding-left: 8px;">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="任务描述">
          <a-textarea placeholder="请输入任务描述" v-decorator="['describe', {}]" :row="3" />
        </a-form-item>
        </a-col>
      </a-row>
    <a-row :gutter="24">
      <a-col :span="16" style="padding-left: 0px;">
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
      </a-col>
     <!-- <a-col :span="12" style="padding-left: 0px;" float:left>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="状态">
          <a-select v-decorator="['state', {}]" placeholder="请选择工单状态">
            <a-select-option value="1">未开始</a-select-option>
            <a-select-option value="2">进行时</a-select-option>
            <a-select-option value="3">已完成</a-select-option>
          </a-select>
        </a-form-item>
      </a-col>-->
    </a-row>

      </a-form>
    </a-spin>

      </a-tab-pane>

      <a-tab-pane tab="工单反馈信息" key="2" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">

            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="客户名称"  :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-auto-complete  disabled="true"  @select="chooseThisA" v-decorator="['companyName', {}]" maxLength="30">
                   <!-- <template slot="dataSource">
                      <a-select-option v-for="item in companyNameListA" :key="item.companyName">{{ item.companyName }}</a-select-option>
                    </template>-->
                  </a-auto-complete>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row >
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划执行时间">
                  <a-date-picker disabled  format="YYYY-MM-DD"  v-decorator="[ 'planExecuTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;" float:left>
                <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="实际执行时间">
                  <a-date-picker disabled format="YYYY-MM-DD"  v-decorator="[ 'realityExecuTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row >
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划完成时间">
                  <a-date-picker disabled  format="YYYY-MM-DD"  v-decorator="[ 'planOutTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际完成时间">
                  <a-date-picker disabled format="YYYY-MM-DD"  v-decorator="[ 'realityOutTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row >
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="计划参与人数">
                  <a-input-number disabled   v-decorator="['planPersonNum', {}]" :min="0" :max="100000"   />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实际参与人数">
                  <a-input-number disabled  v-decorator="['realityPersonNum', {}]" :min="0" :max="100000"  />
                </a-form-item>
              </a-col>
            </a-row>

            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="任务内容" >
                  <a-textarea readonly  v-decorator="['content', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex"/>
                </a-form-item>
              </a-col>
            </a-row>

            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="执行情况" >
                  <a-textarea  readonly  v-decorator="['result', {}]"  :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="客户评价" >
                  <a-select disabled v-decorator="['evaluate', {}]" >
                    <a-select-option value="1">很满意</a-select-option>
                    <a-select-option value="2">满意</a-select-option>
                    <a-select-option value="3">一般</a-select-option>
                    <a-select-option value="4">较差</a-select-option>
                    <a-select-option value="5">很差</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>

            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注" >
                  <a-textarea readonly placeholder="请输入备注"  v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex" />
                </a-form-item>
              </a-col>
            </a-row>


            <a-row>
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <div>
                    <div v-for="(item,index) in model.filelist1" :key="index">{{item.fileName}}</div>
                  </div>
                </a-form-item>
              </a-col>
            </a-row>

          </a-form>
        </a-spin>
      </a-tab-pane>
    </a-tabs>
    <AddProjectItemRel ref="addProjectItemRel" @func="modalFormOk"></AddProjectItemRel>


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
  import AddProjectItemRel from "./AddProjectItemRel1";



  export default {
    name: "addWorkOrderInfo",
    components: {AddProjectItemRel, ATextarea},
    data () {
      return {
        title:"操作",
        visible: false,
        defaultActiveKey: '1',
        model: {},
        prjItemId: '',
        isUpload: false,
        formData: {},
        chargePersonList:[],
        prjItemNamesList:[],
        Id:"",
        fileList:[],
        fileList1:[],
        prjItemNames:[],
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
        isOpen: false,
        isOpen1: false,
        chooseProjectItemName: '',
        chooseChargePersonName:'',
        optionVal: '',
        optionVal1: '',
        textVal: '',
        url: {
          add: "/renche/workOrder/addWorkOrderInfo",
          edit: "/renche/workOrder/up",
          getn:"/renche/workOrder/getn",
          //fileUpload: doMian + "/sys/common/upload",
          fileUpload: "/sys/common/upload",
          searchSysUser:"/renche/workOrder/userList",
          worSerlist: "/renche/WorkSerivice/qryWorkSerivice",
        },
      }
    },
    created () {
      //初始化工程点名稱列表
      this.initprojectNames();
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {


      callback: function(key){
       /* if(key != 1){
          if(this.model.contractId != undefined && this.model.contractId != ""){
            this.defaultActiveKey = key;
          }else{
            this.$message.warning("请先创建合同信息");
          }
        }else{
          this.defaultActiveKey = key;
        }*/
        this.defaultActiveKey = key;
      },

      getChargePersonList: function(val){

        if(val != this.chooseChargePersonName) {

          this.chooseChargePersonName = '';

          getAction(this.url.searchSysUser, {pageNo: "1", pageSize: "10", name: val}).then((res) => {
            if (res.success) {
              console.log(res.result.list)
              this.chargePersonList = res.result.list;
              this.isOpen1 = true;
              if (this.chargePersonList.length == 1) {
                this.model.chargePerson = this.chargePersonList[0].username;
              } else {
                this.model.chargePerson = "";
              }
            }
          })
        }
      },
      chooseThisA: function(val){
        this.model.chargePerson= val;
        this.isOpen1 = false;
        this.chooseChargePersonName = val;

      },
      chooseThis: function(val){
        this.model.chargePerson= val;


      },

      getPrjItemNamesList: function(val){
        if(val != undefined && val != this.chooseProjectItemName) {
          this.textVal = val;
          this.prjItemId = '';
          this.chooseProjectItemName = '';

          getAction(this.url.getn, {pageNo: "1", pageSize: "10", name: val}).then((res) => {
            if (res.success) {
              console.log(res.result.list)
              this.prjItemNamesList = res.result.list;
              this.isOpen = true;
              if (this.prjItemNamesList.length == 1) {
                this.model.prjItemName = this.chargePersonList[0].prjItemName;
              } else if(this.prjItemNamesList.length == 0){
                this.prjItemId = "";
              }else{
                this.isOpen = true;
              }
            }
          })
        }
      },
      chooseThisB: function(val,option){
       // this.model.prjItemName= val;
        this.isOpen = false;
       // this.chooseProjectItemName = val;
        if(val != 'close'){
          this.prjItemId = option.key;
          this.optionVal = val;
          this.chooseProjectItemName = val;
          this.prjItemNamesList = [];
        }else{
          this.optionVal = this.textVal;
        }
      },




      beforeUpload: function (file) {
       /* var fileType = file.type;
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
        const workNam = this.model.workName;
        var fileName=data.file.name.toString();
        var str1=fileName.substring(0,fileName.lastIndexOf("."));
        var str2=fileName.substring(fileName.lastIndexOf("."), fileName.length);
        const copyFile = new File([data.file], `${workNam}_${nowDate}_${str1}_${timeStamp}_${str2}`)
        console.log(copyFile)
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
      //初始化工程点名稱列表
      initprojectNames(){
        var that=this;
        getAction(this.url.getn).then((res)=> {

          if (res.success) {

            this.prjItemNames = res.result;
          } else {
            that.$message.warning(res.message);
          }
        })

      },

      add () {

        this.edit({});

      },
      edit (record) {

        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        this.avatar = record.fileRelId1 == undefined?'':record.fileRelId1;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        if(record.filelist1 == undefined){
          record.filelist1 = [];
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.fileList = this.model.filelist;
        this.visible = true;
        this.isUpload = false;
        this.fileList = [];
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'workName','createPerson','describe','chargePerson','content','prjItemName','completeTime','status','planExecuTime','companyName','realityExecuTime','planOutTime','realityOutTime','planPersonNum','realityPersonNum','result','evaluate','remark'));
          //时间格式化
          this.form.setFieldsValue({completeTime:this.model.completeTime?moment(this.model.completeTime,'YYYY-MM-DD'):null});
          //时间格式化
          //  this.form.setFieldsValue({visitTime:this.model.visitTime?moment(this.model.visitTime,'YYYY-MM-DD HH:mm:ss'):null});
          this.form.setFieldsValue({planExecuTime:this.model.planExecuTime?moment(this.model.planExecuTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({realityExecuTime:this.model.realityExecuTime?moment(this.model.realityExecuTime,'YYYY-MM-DD '):null});
          this.form.setFieldsValue({planOutTime:this.model.planOutTime?moment(this.model.planOutTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({realityOutTime:this.model.realityOutTime?moment(this.model.realityOutTime,'YYYY-MM-DD'):null});


        });

      },
      close () {
        this.$emit('close');
        this.isOpen = false;
        this.chooseProjectItemName='';
        this.isOpen1 = false;
        this.chooseChargePersonName='';
        this.visible = false;
      },
      handleOk () {

        const that = this;
        that.isOpen = false;
        if(that.prjItemId == ''){
          this.form.setFieldsValue({prjItemName:""})
        }
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.workId){

              httpurl+=this.url.add;
              method = 'post';
            }else{

              httpurl+=this.url.edit;
              method = 'put';
            }

            if(this.avatar!=null && this.avatar != "" && this.avatar !=undefined) {
              let a = this.avatar.charAt(this.avatar.length - 1);
              if (a == ",") {
                this.avatar = this.avatar.substring(0, this.avatar.length - 1);
              }
            }
            this.model.fileRelId = this.avatar;
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.completeTime = formData.completeTime?formData.completeTime.format('YYYY-MM-DD'):null;
            httpAction(httpurl,formData,method).then((res)=>{
              var meth = method;
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
        this.$emit('ok');
        this.defaultActiveKey = "1";
        this.close()
      },
      hideDownList(){
        this.isOpen = false;
      },
      hideDownList1(){
        this.isOpen1 = false;
      },
      modalFormOk(data) {
        this.prjItemId = data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
      },
      showProjectItem:function (){
        this.$refs.addProjectItemRel.show(this.model.contractId);
        this.$refs.addProjectItemRel.title = "选择添加关联工程点信息";
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