<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="任务信息" key="1" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="任务名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-textarea placeholder="请输入任务名称" v-decorator="['taskName', {rules: [{ required: true,message: '请输入任务名称' }]}]" :autosize="{ minRows: 1, maxRows: 2 }" maxLength="150"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="联系人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入联系人" v-decorator="[ 'contactPerson', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="联系电话" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入联系电话" v-decorator="['contactTel', {}]"  />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="任务内容" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-textarea placeholder="请输入任务内容" v-decorator="['taskContent', {rules: [{ required: true,message: '请输入任务内容' }]}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="500"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="负责人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-select mode="multiple" style="width: 100%" placeholder="请选择负责人" v-decorator="['selectUser', {rules: [{ required: true,message: '请选择负责人' }]}]" @change="changUser">
                    <a-select-option v-for="(user,index) in userList" :key="user.realname" :value="user.id">
                      {{ user.realname }}
                    </a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="16" style="padding-left: 0px;">
                <a-form-item label="关联工程点" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-auto-complete :open="isOpen" :optionLabelProp="optionVal"  @select="chooseThis" @blur="getPrjItemList" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确工程点名称', }]}]" maxLength="150">
                    <a-textarea placeholder="请输入工程点名称" class="custom" :autosize="{ minRows: 1, maxRows: 2 }" />
                    <template slot="dataSource">
                      <a-select-option key="close">
                          <a-icon @click="hideDownList()"  type="close" style="float: right;"></a-icon>
                      </a-select-option>
                      <a-select-option v-for="item in itemList" :key="item.prjItemId" :value="item.prjItemName">{{ item.prjItemName }}</a-select-option>
                    </template>
                  </a-auto-complete>
                </a-form-item>
              </a-col>
              <a-col>
                <a-button @click="handleAddPrjItem" type="primary" icon="plus">新增</a-button>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="计划开始时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="[ 'planStartTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="计划结束时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="[ 'planEndTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="实际开始时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="[ 'startTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="实际结束时间" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="[ 'endTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="16" style="padding-left: 0px;">
                <a-form-item label="任务进度" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-textarea placeholder="请输入任务进度" v-decorator="['progressOfItem', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="20" style="padding-left: 0px;">
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
      <a-tab-pane tab="需要设备清单" key="2" style="padding-bottom: 14px" v-if="model.isMakeDemand == '1'">
        <div style="padding-top: 42px;">
          <a-table
            ref="table"
            bordered
            rowKey="demandId"
            :columns="columnsRead"
            :dataSource="dataSource"
            :loading="loading"
            :scroll="{ y: 500 }"
            style="padding-top: 10px;">
          </a-table>
        </div>
      </a-tab-pane>
      <a-tab-pane tab="需要设备清单" key="2" style="padding-bottom: 14px" v-if="model.isMakeDemand != '1'">
        <div style="float: right;">
          <a-button type="primary" @click="addDemand" icon="plus" >新增</a-button>
        </div>
        <div style="padding-top: 42px;">
          <a-table
            ref="table"
            bordered
            rowKey="demandId"
            :columns="columns"
            :dataSource="dataSource"
            :loading="loading"
            :scroll="{ y: 500 }"
            style="padding-top: 10px;">

                <span slot="action" slot-scope="text, record">
                  <a @click="handleEdit(record)">编辑</a>
                  <a-divider type="vertical"/>
                  <a @click="handleDelete(record.demandId)">删除</a>
                </span>
          </a-table>
        </div>
      </a-tab-pane>
    </a-tabs>

    <AddProjectItem ref="addProjectItem" @ok="modalFormOk"></AddProjectItem>
    <DemandModel ref="demandModel" @ok="addDemandOk"></DemandModel>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
      <a-button type="primary" @click="handleOk('0')" v-if="model.status != '1' || model.status != '2'">保存</a-button>
      <a-button type="primary" @click="handleOk('1')">提交</a-button>
    </div>
  </a-modal>
</template>

<script>
  import {getAction, httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import ACol from "ant-design-vue/es/grid/Col";
  import AddProjectItem from "./AddProjectItem";
  import DemandModel from "./DemandModel";
  import Error from "../../../result/Error";

  export default {
    name: "taskInfoModel",
    components: {DemandModel, ACol, ATextarea,AddProjectItem},
    data () {
      return {
        title:"操作",
        visible: false,
        isUpload: false,
        isOpen: false,
        prjItemId: '',
        choosePrjItemName: '',
        optionVal: '',
        textVal: '',
        formData: {},
        model: {},
        itemList: [],
        fileList:[],
        selectUser: [],
        selectUserName: [],
        userList: [],
        defaultActiveKey: "1",
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
          sm: { span: 4 },
        },
        filewrapperCol: {
          xs: { span: 24 },
          sm: { span: 19 },
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
            dataIndex: 'equipmentName',
            width: 190,
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipmentModel',
            width: 190,
          },
          {
            title: '需要数量',
            align: "center",
            dataIndex: 'equipmentNumber',
            width: 100,
          },
          {
            title: '拥有方式',
            align: "center",
            dataIndex: 'haveWay',
            width: 100,
            customRender: (text, record, index) => {
              //字典值替换通用方法
              if (text == '0'){
                return "租赁";
              }else if(text == '1'){
                return "购买";
              }
            }
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'remarks',
            width: 220,
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }

        ],
        //表头
        columnsRead:[
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
            dataIndex: 'equipmentName',
            width: 190,
          },
          {
            title: '设备型号',
            align: "center",
            dataIndex: 'equipmentModel',
            width: 190,
          },
          {
            title: '需要数量',
            align: "center",
            dataIndex: 'equipmentNumber',
            width: 100,
          },
          {
            title: '拥有方式',
            align: "center",
            dataIndex: 'haveWay',
            width: 100,
            customRender: (text, record, index) => {
              //字典值替换通用方法
              if (text == '0'){
                return "租赁";
              }else if(text == '1'){
                return "购买";
              }
            }
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'remarks',
            width: 220,
          }
        ],
        //数据集
        dataSource: [],
        loading: false,
        uploadLoading: false,
        headers: {},
        avatar: "",
        isArris: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          queryNaList: "/sys/user/queryNaList",
          add: "/renche/taskInfo/add",
          edit: "/renche/taskInfo/edit",
          fileUpload: "/sys/common/upload",
          searchPrjItem: "/renche/projectItem/queryItemList",
          demandList: "/renche/demand/queryDemandList",
          taskDemandList: "/renche/demand/queryDemandList",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    methods: {
      loadUserList(){
        getAction(this.url.queryNaList).then((res) => {
          if (res.success) {
            this.userList = res.result;
          }
        })
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
      add () {
        this.edit({});
      },
      edit (record) {
        this.loadUserList();
        this.avatar = record.fileRelId == undefined?'':record.fileRelId;
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.fileList = [];

        if(record.receiveUser != undefined && record.receiveUser != null && record.receiveUser != ''){
          this.selectUser = record.receiveUser.split(', ');
          this.selectUserName = record.receiveUserName.split(', ');
        }

        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'taskName','contactPerson','contactTel','taskContent','prjItemName','progressOfItem'))
          this.form.setFieldsValue({selectUser:this.selectUser});
          //时间格式化
          this.form.setFieldsValue({planStartTime:this.model.planStartTime?moment(this.model.planStartTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({planEndTime:this.model.planEndTime?moment(this.model.planEndTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({startTime:this.model.startTime?moment(this.model.startTime,'YYYY-MM-DD'):null});
          this.form.setFieldsValue({endTime:this.model.endTime?moment(this.model.endTime,'YYYY-MM-DD'):null});
        });

        if(this.model.taskId){
          this.loadDemand(this.model.taskId);
        }

      },
      close () {
        this.$emit('close');
        this.isOpen = false;
        this.defaultActiveKey = "1";
        this.dataSource = [];
        this.selectUser = [];
        this.selectUserName = [];
        this.visible = false;
      },
      handleOk (status) {
        const that = this;
        // 触发表单验证
        that.isOpen = false;
        if(that.prjItemId == ''){
          this.form.setFieldsValue({prjItemName:""})
        }
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.taskId){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }

            that.model.fileRelId = that.avatar;
            if(that.prjItemId != ''){
              that.model.prjItemId = that.prjItemId;
            }
            that.model.status = status;
            let formData = Object.assign(that.model, values);
            formData.demandList = this.dataSource;
            formData.selectUserName = this.selectUserName;
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
        this.$emit('ok');
        this.close();
      },
      getPrjItemList(val){
        if(val != undefined && val != this.choosePrjItemName){
          this.textVal = val;
          this.prjItemId = '';
          this.choosePrjItemName = '';
          getAction(this.url.searchPrjItem, {prjItemName: val,pageNo: "1",pageSize: "30",}).then((res) => {
            if (res.success) {
              this.itemList = res.result.list;
              if(this.itemList.length == 0){
                this.prjItemId = "";
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
          this.prjItemId = option.key;
          this.optionVal = val;
          this.choosePrjItemName = val;
          this.itemList = [];
        }else{
          this.optionVal = this.textVal;
        }
      },
      handleAddPrjItem: function () {
        this.isOpen = false;
        this.$refs.addProjectItem.add();
        this.$refs.addProjectItem.title = "新增";
      },
      modalFormOk(data){
        this.prjItemId = data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
      },
      callback: function(key){
          this.defaultActiveKey = key;
      },
      addDemand(){
        var record = {};
        record.demandList = this.dataSource;
        this.$refs.demandModel.add(record);
        this.$refs.demandModel.title = "添加需要设备";
      },
      handleEdit(record){
        var info = Object.assign({}, record);
        info.demandList = this.dataSource;
        this.$refs.demandModel.add(info);
        this.$refs.demandModel.title = "修改需要设备";
      },
      addDemandOk(data){
        this.dataSource = data;
      },
      changUser(val,option){
        this.selectUserName = [];
        option.some((item,i) => {
          this.selectUserName.push(item.key);
        })
      },
      loadDemand(taskId){
        getAction(this.url.taskDemandList, {taskId: taskId}).then((res) => {
          if (res.success) {
            this.dataSource = res.result;
          }
        })
      },
      handleDelete: function (id) {
        this.dataSource.some((item,i) => {
          if(item.demandId == id){
            this.dataSource.splice(i);
          }
        })
      },
      hideDownList(){
        this.isOpen = false;
      },
      // checkPrjItemId(rule, value, callback){
      //   if(this.prjItemId == ''){
      //     callback(new Error("请输入已创建的工程点名称"));
      //   }
      // },
    }
  }
</script>

<style scoped>

</style>