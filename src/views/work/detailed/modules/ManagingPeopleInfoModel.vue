<template>
  <a-modal
    :title="title"
    :width="1080"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="人员管理系统部署验收信息" key="1" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row >
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="关联工程点" >
                  <a-input placeholder="请输入工程名称" v-decorator="['prjItemName', {rules: [{ required: true, message: '请输入正确工程点名称', }]}]" @click="showProjectItem" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="验收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['checkTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="建设单位" >
                  <a-input placeholder="请输入建设单位" v-decorator="['constructionCompany', {}]" @click="showCompanyList"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="监理单位" >
                  <a-input placeholder="请输入监理单位" v-decorator="['supervisorCompany', {}]" @click="showCompanyList1"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">

                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="施工单位" >
                  <a-input placeholder="请输入施工单位" v-decorator="['roadworkCompany', {}]" @click="showCompanyList2"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="实施厂商" >
                  <a-input placeholder="请输入实施厂商" v-decorator="['implementCompany', {}]" @click="showCompanyList3"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="实施厂商" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请签字" v-decorator="['implement', {}]" maxlength="30"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签字日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['implementDate', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="项目施工部" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请签字" v-decorator="['roadwork', {}]" maxlength="30"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签字日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['roadworkDate', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="监理项目部" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请签字" v-decorator="['supervisor', {}]" maxlength="30"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签字日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['supervisorDate', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="业主项目部" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请签字" v-decorator="['user', {}]" maxlength="30"/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签字日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['userDate', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>

          <a-row :gutter="24">
            <a-col :span="16" style="padding-left: 8px;">
              <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="用户意见" >
                <a-textarea  placeholder="请输入用户意见" v-decorator="['userRemark', {}]" :autosize="{ minRows: 2, maxRows: 6 }"   maxLength="1500" />
              </a-form-item>
            </a-col>
          </a-row>

          <a-row style="text-align: center;">
            <a-col>
              <span style="overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="handleOk" icon="check">保存</a-button>
              </span>
            </a-col>
          </a-row>
        </a-spin>
      </a-tab-pane>

      <a-tab-pane tab="软件部署" key="2" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">

          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="软件部署确认项" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-checkbox :indeterminate="indeterminate" :checked="checkAll" @change="onCheckAllChange">
                    全选
                  </a-checkbox>
                  <br />
                  <a-checkbox-group v-model="checkedList" :options="plainOptions" @change="onChange" />
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
        </a-spin>
        <div style="float: right;">
          <a-button @click="addSoftwareDeploy" type="primary" >提交</a-button>
        </div>

      </a-tab-pane>

      <a-tab-pane tab="硬件部署" key="3" style="padding-bottom: 14px">
        <div>
          <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="制证设备部署" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-checkbox :indeterminate="indeterminate1" :checked="checkAll1" @change="onCheckAllChange1">
                    全选
                  </a-checkbox>
                  <br />
                  <a-checkbox-group v-model="checkedList1" :options="plainOptions1" @change="onChange1" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="人员闸机部署" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-checkbox :indeterminate="indeterminate2" :checked="checkAll2" @change="onCheckAllChange2">
                    全选
                  </a-checkbox>
                  <br />
                  <a-checkbox-group v-model="checkedList2" :options="plainOptions2" @change="onChange2" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="车辆道闸部署" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-checkbox :indeterminate="indeterminate3" :checked="checkAll3" @change="onCheckAllChange3">
                    全选
                  </a-checkbox>
                  <br />
                  <a-checkbox-group v-model="checkedList3" :options="plainOptions3" @change="onChange3" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="28">
              <a-col :span="30" style="padding-left: 0px;">
                <a-form-item label="线路工程人员信息移动采集设备" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-checkbox :indeterminate="indeterminate4" :checked="checkAll4" @change="onCheckAllChange4">
                    全选
                  </a-checkbox>
                  <br />
                  <a-checkbox-group v-model="checkedList4" :options="plainOptions4" @change="onChange4" />
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
          </a-spin>
        </div>
        <div>
          <div style="float: right;">
            <a-button @click="addHardwareDeployInfo" type="primary" >提交</a-button>
          </div>
        </div>
      </a-tab-pane>

      <a-tab-pane tab="用户培训" key="4" style="padding-bottom: 14px">
        <div>
          <a-spin :spinning="confirmLoading">

            <a-form :form="form">
              <a-row :gutter="24">
                <a-col :span="16" style="padding-left: 8px;">
                  <a-form-item label="资料移交" :wrapperCol="wrapperCol" :labelCol="labelCol">
                    <a-checkbox :indeterminate="indeterminateA" :checked="checkAllA" @change="onCheckAllChangeA">
                      全选
                    </a-checkbox>
                    <br />
                    <a-checkbox-group v-model="checkedListA" :options="plainOptionsA" @change="onChangeA" />
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row :gutter="28">
                <a-col :span="30" style="padding-left: 0px;">
                  <a-form-item label="系统培训内容" :wrapperCol="wrapperCol" :labelCol="labelCol">
                    <a-checkbox :indeterminate="indeterminateB" :checked="checkAllB" @change="onCheckAllChangeB">
                      全选
                    </a-checkbox>
                    <br />
                    <a-checkbox-group v-model="checkedListB" :options="plainOptionsB" @change="onChangeB" />
                  </a-form-item>
                </a-col>
              </a-row>
            </a-form>
          </a-spin>
        </div>
        <div>
          <div style="float: right;">
            <a-button @click="addUserTrainInfo" type="primary" >提交</a-button>
          </div>
        </div>
      </a-tab-pane>
      <a-tab-pane tab="运行确认" key="5" style="padding-bottom: 14px">
        <div>
          <a-spin :spinning="confirmLoading">

            <a-form :form="form">
              <a-row :gutter="24">
                <a-col :span="16" style="padding-left: 8px;">
                  <a-form-item label="系统运行" :wrapperCol="wrapperCol" :labelCol="labelCol">
                    <a-checkbox :indeterminate="indeterminateC" :checked="checkAllC" @change="onCheckAllChangeC">
                      全选
                    </a-checkbox>
                    <br />
                    <a-checkbox-group v-model="checkedListC" :options="plainOptionsC" @change="onChangeC" />
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row :gutter="28">
                <a-col :span="30" style="padding-left: 0px;">
                  <a-form-item label="硬件运行" :wrapperCol="wrapperCol" :labelCol="labelCol">
                    <a-checkbox :indeterminate="indeterminateD" :checked="checkAllD" @change="onCheckAllChangeD">
                      全选
                    </a-checkbox>
                    <br />
                    <a-checkbox-group v-model="checkedListD" :options="plainOptionsD" @change="onChangeD" />
                  </a-form-item>
                </a-col>
              </a-row>
            </a-form>
          </a-spin>
        </div>
        <div>
          <div style="float: right;">
            <a-button @click="addAffirmRunningInfo" type="primary" >提交</a-button>
          </div>
        </div>
      </a-tab-pane>
    </a-tabs>


    <AddProjectItemRel  ref="addProjectItemRel" @func="addProjectItem"></AddProjectItemRel>
    <company-info-show ref="companyInfoShow" @func="modalFormOk1"></company-info-show>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  const plainOptions = ['一键安装部署正常', '硬件驱动已安装', '开机自启已实现','服务检测正常','在线注册已完成','权限已分配','基本信息已配置','线路工程人员信息移动采集设备（ 线路专用） APP 已部署'];
  const plainOptions1 =['身份证阅读器','摄像头','证卡打印机','','热敏打印机','RFID 写卡器','RFID 卡片','指纹识别仪（ 可选）'];
  const plainOptions2 =['闸机组装','通电检测','进出调试'];
  const plainOptions3 =['闸机组装','通电检测','进出调试（ 车牌识别或刷卡）'];
  const plainOptions4=['移动终端组装'];
  const plainOptionsA=['用户操作手册'];
  const plainOptionsB=['软件功能介绍','制卡（ 办证） 操作','系统升级','系统安装','数据备份','数据恢复','用户答疑'];
  const plainOptionsC=['系统运转正常','正常办理制卡/退卡','接口正常发送/接收闸机数据','定时向基建管理系统上传数据'];
  const plainOptionsD=['人员闸机运行正常','车辆道闸运行正常','人员闸机刷卡(指纹可选)进出正常','车辆道闸车牌识别(刷卡可选)进出正常','线路工程人员信息移动采集设备( 线路专用)刷卡进出正常','线路工程人员信息移动采集设备(线路专用)与人员管理系统数据同步正常'];

  import { httpAction, getAction, deleteAction,postAction} from '@/api/manage'
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import {filterObj} from '@/utils/util'
  import ARow from "ant-design-vue/es/grid/Row";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ACol from "ant-design-vue/es/grid/Col";
  import AddProjectItemRel from "./AddProjectItemRel";
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import CompanyInfoShow from "../../project/show/CompanyInfoShow";


  export default {
    name: "ManagingPeopleInfoModel",
    components: {
      CompanyInfoShow,
      AddProjectItemRel, ACol, ATextarea, ARow,
      },
    data () {
      return {
        title:"操作",
        visible: false,
        isUpload: false,
        formData: {},
        isShow: false,
        model: {},
        elefileList:[],
        fileList:[],
        //字典数组缓存
        typeDictOptions: [],
        remindTypeOptions: [],
        typePrjDictOptions: [],
        companyNameListA: [],
        isOk:true,
        isSearchA: false,
        isSearchB: false,
        isOnFocus: false,
        companyIdA:"",
        companyIdB:"",
        companyIdC:"",
        companyIdD:"",
        thisMessage:"",
        defaultActiveKey: '1',
        checkedList: "" ,
        checkedList1: "" ,
        checkedList2: "" ,
        checkedList3: "" ,
        checkedList4: "" ,
        checkedListA: "" ,
        checkedListB: "" ,
        checkedListC: "" ,
        checkedListD: "" ,
        indeterminate: true,
        indeterminate1: true,
        indeterminate2: true,
        indeterminate3: true,
        indeterminate4: true,
        indeterminateA: true,
        indeterminateB: true,
        indeterminateC: true,
        indeterminateD: true,
        checkAll: false,
        checkAll1:false,
        checkAll2:false,
        checkAll3:false,
        checkAll4:false,
        checkAllA:false,
        checkAllB:false,
        checkAllC:false,
        checkAllD:false,
        plainOptions,
        plainOptions1,
        plainOptions2,
        plainOptions3,
        plainOptions4,
        plainOptionsA,
        plainOptionsB,
        plainOptionsC,
        plainOptionsD,
        softwareDeployId:"",
        hardwareDeployId:"",
        userTrainId:"",
        affirmRuningId:"",
        isfal:"",
        isfal1:"",


        labelCol: {
          xs: { span: 28 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 28 },
          sm: { span: 16 },
        },
        filelabelCol: {
          xs: { span: 28 },
          sm: { span: 3 },
        },
        filewrapperCol: {
          xs: { span: 28 },
          sm: { span: 21 },
        },
        uploadLoading: false,
        headers: {},
        avatar: "",
        avatarElec: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },

        //数据集
        dataSource: [],
        // 分页参数
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '15', '20'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        isorter: {
          column: 'createTime',
          order: 'desc',
        },
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],


        backIsorter: {
          column: 'createTime',
          order: 'desc',
        },
        backLoading: false,
        backSelectedRowKeys: [],
        backSelectedRows: [],
        isOpen: false,
        isOpen1: false,
        chooseCompanyNameA:'',
        chooseCompanyNameB:'',
        optionVal: '',
        textVal: '',
        optionVal1: '',
        textVal1: '',
        url: {
          add: "/renche/managingPeople/add",
          edit: "/renche/managingPeople/edit",
          searchCompany: "/renche/companyInfo/searchCompany",
          softwareDeployList:"/renche/softwareDeploy/softwareDeployList",
          softwareDeployAdd:"/renche/softwareDeploy/add",
          softwareDeployEdit:"/renche/softwareDeploy/edit",
          hardwareDeployAdd:"renche/hardwareDeployInfo/add",
          hardwareDeployList:"renche/hardwareDeployInfo/list",
          hardwareDeployEdit:"renche/hardwareDeployInfo/edit",
          userTrainInfoList:"renche/userTrainInfo/list",
          userTrainInfoAdd:"renche/userTrainInfo/add",
          userTrainInfoEdit:"renche/userTrainInfo/edit",
          affirmRunningInfoList:"renche/affirmRunningInfo/list",
          affirmRunningInfoAdd:"renche/affirmRunningInfo/add",
          affirmRunningInfoEdit:"renche/affirmRunningInfo/edit"
,
          fileUpload: "/sys/common/upload",
          invociList: "renche/invoci/qryInvociList",
          filelist: "/renche/purchase/fileList",
          backList: "/renche/moneyBack/list",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
      //初始化字典配置
      this.initDictConfig();
    },
    methods: {
      initDictConfig() {

        //初始化字典 - 工程类型
        initDictOptions('PROJECTITEMTYPE').then((res) => {
          if (res.success) {
            this.typePrjDictOptions = res.result;
          }
        });
      },
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.prjItemList, params).then((res) => {
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
        param.contractId = this.model.contractId;
        return filterObj(param);
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
        this.model = Object.assign({}, record);
        this.visible = true;

        if(this.model.managingPeopleId != null && this.model.managingPeopleId != undefined){
          this.isShow = true;
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model,'prjItemName','checkTime','constructionCompany','supervisorCompany','roadworkCompany','implementCompany','implement','implementDate','roadworkDate','supervisor','supervisorDate','user','userDate','userRemark'))
            this.form.setFieldsValue({checkTime:this.model.checkTime?moment(this.model.checkTime):null});
            this.form.setFieldsValue({implementDate:this.model.implementDate?moment(this.model.implementDate):null});
            this.form.setFieldsValue({roadworkDate:this.model.roadworkDate?moment(this.model.roadworkDate):null});
            this.form.setFieldsValue({supervisorDate:this.model.supervisorDate?moment(this.model.supervisorDate):null});
            this.form.setFieldsValue({userDate:this.model.userDate?moment(this.model.userDate):null});
          });








        }
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.isOpen = false;
        this.chooseCompanyNameA = '';
        this.isOpen1 = false;
        this.chooseCompanyNameB = '';
      },
      handleOk () {
        const that = this;
        that.isOpen = false;
        that.isOpen1 = false;
        if(that.companyIdA == ""){
          this.form.setFieldsValue({constructionCompany:""});
        }
        if(that.companyIdB == ""){
          this.form.setFieldsValue({supervisorCompany:""});
        }
        if(that.companyIdC == ""){
          this.form.setFieldsValue({roadworkCompany:""});
        }
        if(that.companyIdD == ""){
          this.form.setFieldsValue({implementCompany:""});
        }
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            if(that.isOk){
              that.confirmLoading = true;
              let httpurl = '';
              let method = '';
              if(!that.model.managingPeopleId){
                httpurl+=this.url.add;
                method = 'post';
              }else{
                httpurl+=this.url.edit;
                method = 'put';
              }

              this.model.fileRelId = that.avatar;
              this.model.elecFileRel = that.avatarElec;

              let formData = Object.assign(this.model, values);

              httpAction(httpurl,formData,method).then((res)=>{
                if(res.success){
                  if(res.result != null){
                    this.model.managingPeopleId = res.result.managingPeopleId;
                  }
                  that.$message.success(res.message);
                }else{
                  that.$message.warning(res.message);
                }
              }).finally(() => {
                that.confirmLoading = false;
              })
            }else{
              that.$message.warning(this.thisMessage);
              that.thisMessage = "";
            }
          }
        })
      },
      handleCancel () {
        this.$emit('ok');
        this.defaultActiveKey = "1";
        this.close()
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


      modalFormOk1(data) {
        debug;
        if(data.mark == '1'){
          this.companyIdA = data.companyId;
          this.form.setFieldsValue({constructionCompany:data.companyName});
        }else if(data.mark == '2'){
          this.companyIdB = data.companyId;
          this.form.setFieldsValue({supervisorCompany:data.companyName});
        }else if(data.mark == '3'){
          this.companyIdC = data.companyId;
          this.form.setFieldsValue({roadworkCompany:data.companyName});
        }else if(data.mark == '4'){
          this.companyIdD = data.companyId;
          this.form.setFieldsValue({implementCompany:data.companyName});
        }

      },
      callback: function(key){
        this.loadMan();
        if(key != 1){
          if(this.model.managingPeopleId != undefined && this.model.managingPeopleId != ""){
            this.defaultActiveKey = key;
          }else{
            this.$message.warning("请先创建验收确认单信息");
            return;
          }
        }else{
          this.defaultActiveKey = key;
        }
        this.defaultActiveKey = key;

      },
      showProjectItem:function (){
        this.$refs.addProjectItemRel.show(this.model.contractId);
        this.$refs.addProjectItemRel.title = "选择添加关联工程点信息";
      },
      addProjectItem(data) {
        this.prjItemId=data.prjItemId;
        this.form.setFieldsValue({prjItemName:data.prjItemName});
        this.loadData();
      },
      addSoftwareDeploy(){
        const that = this;
        let httpurl = '';
        let method = '';
        if(!that.softwareDeployId){
          httpurl+=this.url.softwareDeployAdd;
          method = 'post';
        }else{
          httpurl+=this.url.softwareDeployEdit;
          method = 'put';
        }
        let data={
          softwareDeployId:that.softwareDeployId,
          managingPeopleId:that.model.managingPeopleId ,
          checkedList: that.checkedList,
        }

            httpAction(httpurl,data,method).then((res)=>{
              if(res.success){
                  that.$message.success(res.message);
              }else{
                that.$message.warning(res.message);
              }
            })


      },

      addHardwareDeployInfo(){
        const that = this;
        let httpurl = '';
        let method = '';
        if(!that.hardwareDeployId){
          httpurl+=this.url.hardwareDeployAdd;
          method = 'post';
        }else{
          httpurl+=this.url.hardwareDeployEdit;
          method = 'put';
        }
        let data={
          hardwareDeployId:that.hardwareDeployId,
          managingPeopleId:that.model.managingPeopleId ,
          checkedList1: that.checkedList1,
          checkedList2: that.checkedList2,
          checkedList3: that.checkedList3,
          checkedList4: that.checkedList4,

        }

        httpAction(httpurl,data,method).then((res)=>{
          if(res.success){
            that.$message.success(res.message);
          }else{
            that.$message.warning(res.message);
          }
        })
      },


      addUserTrainInfo(){

        const that = this;
        let httpurl = '';
        let method = '';
        alert("that.userTrainId:"+that.userTrainId);
        if(!that.userTrainId){
          httpurl+=this.url.userTrainInfoAdd;
          method = 'post';
        }else{
          httpurl+=this.url.userTrainInfoEdit;
          method = 'put';
        }
        let data={
          userTrainId:that.userTrainId,
          managingPeopleId:that.model.managingPeopleId ,
          checkedListA: that.checkedListA,
          checkedListB: that.checkedListB,

        }

        httpAction(httpurl,data,method).then((res)=>{
          if(res.success){
            that.$message.success(res.message);
          }else{
            that.$message.warning(res.message);
          }
        })

      },


      addAffirmRunningInfo(){
        const that = this;
        let httpurl = '';
        let method = '';
        /*alert("that.userTrainId:"+that.affirmRuningId);*/
      /*  alert("that.model.managingPeopleId "+that.model.managingPeopleId )*/
        if(!that.affirmRuningId){
          httpurl+=this.url.affirmRunningInfoAdd;
          method = 'post';
        }else{
          httpurl+=this.url.affirmRunningInfoEdit;
          method = 'put';
        };
        let data={
          affirmRuningId:that.affirmRuningId,
          managingPeopleId:that.model.managingPeopleId ,
          checkedListC: that.checkedListC,
          checkedListD: that.checkedListD,

        };
     /*   alert("httpurl:"+httpurl);*/

        httpAction(httpurl,data,method).then((res)=>{
          if(res.success){
            that.$message.success(res.message);
          }else{
            that.$message.warning(res.message);
          }
        })

      },



/*

      handleDel: function (id) {
        var that = this;
        var managingPeopleId = this.model.managingPeopleId;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delprjItem, {prjItemId: id, managingPeopleId: managingPeopleId}).then((res) => {
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
*/




      showCompanyList:function (){
        var record = {};
        record.mark = '1';
        this.$refs.companyInfoShow.show(record);
        this.$refs.companyInfoShow.title = "选择公司信息";
      },
      showCompanyList1:function (){
        var record = {};
        record.mark = '2';
        this.$refs.companyInfoShow.show(record);
        this.$refs.companyInfoShow.title = "选择公司信息";
      },
      showCompanyList2:function (){
        var record = {};
        record.mark = '3';
        this.$refs.companyInfoShow.show(record);
        this.$refs.companyInfoShow.title = "选择公司信息";
      },
      showCompanyList3:function (){
        var record = {};
        record.mark = '4';
        this.$refs.companyInfoShow.show(record);
        this.$refs.companyInfoShow.title = "选择公司信息";
      },
      onChange(checkedList) {
        this.indeterminate = !!checkedList.length && checkedList.length < plainOptions.length;
        this.checkAll = checkedList.length === plainOptions.length;
      },
      onCheckAllChange(e) {
        Object.assign(this, {
          checkedList: e.target.checked ? plainOptions : [],
          indeterminate: false,
          checkAll: e.target.checked,
        });
      },
      onChange1(checkedList1) {
        this.indeterminate1 = !!checkedList1.length && checkedList1.length < plainOptions1.length;
        this.checkAll1 = checkedList1.length === plainOptions1.length;
      },
      onCheckAllChange1(e) {
        Object.assign(this, {
          checkedList1: e.target.checked ? plainOptions1 : [],
          indeterminate1: false,
          checkAll1: e.target.checked,
        });
      },
      onChange2(checkedList2) {
        this.indeterminate2 = !!checkedList2.length && checkedList2.length < plainOptions2.length;
        this.checkAll2= checkedList2.length === plainOptions2.length;
      },
      onCheckAllChange2(e) {
        Object.assign(this, {
          checkedList2: e.target.checked ? plainOptions2 : [],
          indeterminate2: false,
          checkAll2: e.target.checked,
        });
      },
      onChange3(checkedList3) {
        this.indeterminate3 = !!checkedList3.length && checkedList3.length < plainOptions3.length;
        this.checkAll2= checkedList3.length === plainOptions3.length;
      },
      onCheckAllChange3(e) {
        Object.assign(this, {
          checkedList3: e.target.checked ? plainOptions3 : [],
          indeterminate3: false,
          checkAll3: e.target.checked,
        });
      },

      onChange4(checkedList4) {
        this.indeterminate4 = !!checkedList4.length && checkedList4.length < plainOptions4.length;
        this.checkAll4= checkedList4.length === plainOptions4.length;
      },
      onCheckAllChange4(e) {
        Object.assign(this, {
          checkedList4: e.target.checked ? plainOptions4 : [],
          indeterminate4: false,
          checkAll2: e.target.checked,
        });
      },

      onChangeA(checkedListA) {
        this.indeterminateA = !!checkedListA.length && checkedListA.length < plainOptionsA.length;
        this.checkAllA= checkedListA.length === plainOptionsA.length;
      },
      onCheckAllChangeA(e) {
        Object.assign(this, {
          checkedListA: e.target.checked ? plainOptionsA : [],
          indeterminateA: false,
          checkAllA: e.target.checked,
        });
      },

      onChangeB(checkedListB) {
        this.indeterminateB = !!checkedListB.length && checkedListB.length < plainOptionsB.length;
        this.checkAllB= checkedListB.length === plainOptionsB.length;
      },
      onCheckAllChangeB(e) {
        Object.assign(this, {
          checkedListB: e.target.checked ? plainOptionsB : [],
          indeterminateB: false,
          checkAllB: e.target.checked,
        });
      },

      onChangeC(checkedListC) {
        this.indeterminateC = !!checkedListC.length && checkedListC.length < plainOptionsC.length;
        this.checkAllA= checkedListC.length === plainOptionsC.length;
      },
      onCheckAllChangeC(e) {
        Object.assign(this, {
          checkedListC: e.target.checked ? plainOptionsC : [],
          indeterminateA: false,
          checkAllC: e.target.checked,
        });
      },

      onChangeD(checkedListD) {
        this.indeterminateD = !!checkedListD.length && checkedListD.length < plainOptionsD.length;
        this.checkAllD= checkedListD.length === plainOptionsD.length;
      },
      onCheckAllChangeD(e) {
        Object.assign(this, {
          checkedListD: e.target.checked ? plainOptionsD : [],
          indeterminateB: false,
          checkAllD: e.target.checked,
        });
      },


      loadMan(){

        getAction(this.url.softwareDeployList, {managingPeopleId: this.model.managingPeopleId}).then((res) => {
          if (res.success) {
            this.checkedList = res.result.softwareDeployName;
            this.softwareDeployId=res.result.softwareDeployId;

          }

        });
        getAction(this.url.hardwareDeployList, {managingPeopleId: this.model.managingPeopleId}).then((res) => {
          if (res.success) {
            this.checkedList1 = res.result.madeCertificate;
            this.checkedList2 = res.result.personnelGate;
            this.checkedList3 = res.result.carGate;
            this.checkedList4 = res.result.moveEquipment;
            this.hardwareDeployId=res.result.hardwareDeployId;

          }
        });
        getAction(this.url.userTrainInfoList, {managingPeopleId: this.model.managingPeopleId}).then((res) => {
          if (res.success) {
            this.checkedListA = res.result.dataTransfer;
            this.checkedListB = res.result.trainContent;
            this.userTrainId = res.result.userTrainId;
          }
        });
        getAction(this.url.affirmRunningInfoList, {managingPeopleId: this.model.managingPeopleId}).then((res) => {
          if (res.success) {
            this.checkedListC = res.result.sysRun;
            this.checkedListD = res.result.hardwareRun;
            this.affirmRuningId = res.result.affirmRuningId;
          }
        });


      },





    }
  }
</script>

<style scoped>
</style>