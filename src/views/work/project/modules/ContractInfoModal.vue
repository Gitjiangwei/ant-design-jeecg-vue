<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleCancel"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="合同信息" key="1" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="合同名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入合同名称" v-decorator="['contractName', {rules: [{ required: true, message: '请输入合同名称', }]}]"  maxLength="150" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="甲方公司" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-auto-complete placeholder="请输入甲方公司"  @search="getCompanyListA" @select="chooseThisA" v-decorator="['companyNameA', {rules: [{ required: true, message: '请输入甲方公司', }]}]" maxLength="30">
                    <template slot="dataSource">
                      <a-select-option v-for="item in companyNameListA" :key="item.companyId">{{ item.companyName }}</a-select-option>
                    </template>
                  </a-auto-complete>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="甲方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入甲方合同编号" v-decorator="['contractNoA', {rules: [{ required: false, message: '请输入甲方合同编号', }]}]"  maxLength="30"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="乙方公司" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-auto-complete placeholder="请输入乙方公司" @search="getCompanyListB" @select="chooseThisB" v-decorator="['companyNameB', {rules: [{ required: true, message: '请输入乙方公司', }]}]"  maxLength="30">
                    <template slot="dataSource">
                      <a-select-option v-for="item in companyNameListB" :key="item.companyId">{{ item.companyName }}</a-select-option>
                    </template>
                  </a-auto-complete>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="乙方合同编号" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入乙方合同编号" v-decorator="['contractNoB', {rules: [{ required: false, message: '请输入乙方合同编号', }]}]"  maxLength="30"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="合同类型" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-select v-decorator="['contractType', {rules: [{ required: true, message: '请选择合同类型', }]}]" placeholder="请选择合同类型">
                    <a-select-option value="">请选择</a-select-option>
                    <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="合同金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input-number v-decorator="['contractMoney', {rules: [{ required: true,message: '请输入正确合同金额' }],initialValue: '0'}]" :min="0" :max="99999999999" :step="0.01" style="width: 60%;" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="合同状态" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-select v-decorator="['contractStatus', {rules: [{ required: true, message: '请选择合同签订状态', }]}]" placeholder="请选择合同签订状态">
                    <a-select-option value="">请选择</a-select-option>
                    <a-select-option value="2">未签订</a-select-option>
                    <a-select-option value="1">已签订</a-select-option>
                    <a-select-option value="0">已结束</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签订日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['signingTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="结束日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker placeholder="请输入结束日期" v-decorator="['overTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker placeholder="请输入签收日期" v-decorator="['signInTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="甲方经办人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入甲方经办人" v-decorator="['operatorA', {}]"  maxLength="20" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="乙方经办人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入乙方经办人" v-decorator="['operatorB', {}]"  maxLength="20" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row v-show="isShow">
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="发票金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input v-decorator="['allReturnMoney', {}]" disabled />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="发票占比" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input v-decorator="['returnMoneyPercent', {}]"  disabled />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row v-show="isShow">
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="回款金额" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input v-decorator="['allReturnMoney', {}]" disabled />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="回款占比" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input v-decorator="['returnMoneyPercent', {}]"  disabled />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="电子版附件"
                  hasFeedback>
                  <!--  -->
                  <a-upload
                    name="file"
                    :multiple="true"
                    :action="uploadAction"
                    :headers="headers"
                    @change="handleChangeElec"
                  >
                    <a-button>
                      <a-icon type="upload"/>
                      上传
                    </a-button>
                  </a-upload>
                  <div>
                    <div v-for="(item,index) in model.elefilelist" :key="index">{{item.fileName}}</div>
                  </div>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="扫描件附件"
                  hasFeedback>
                  <!--  -->
                  <a-upload
                    name="file"
                    :multiple="true"
                    :action="uploadAction"
                    :headers="headers"
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
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="提醒周期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-select v-decorator="['remindPeriodType', {}]" placeholder="请选择提醒周期类型">
                    <a-select-option value="">请选择</a-select-option>
                    <a-select-option v-for="item in remindTypeOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="关联招标" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入招标信息" v-decorator="['prjName', {}]" @click="showTender"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="备注" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-textarea placeholder="请输入备注" v-decorator="['remark', {}]" :autosize="{ minRows: 2, maxRows: 6 }" maxLength="1500"/>
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

      <a-tab-pane tab="关联工程点" key="2" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px" @click="">
        <div style="float: right;">
          <a-button @click="showProjectItem" type="primary" icon="plus">新增</a-button>
        </div>

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
              <a @click="handleDel(record.prjItemId)">删除</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>

      <a-tab-pane tab="发票信息" key="3" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
        <div>
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="invociId"
            :columns="invociColumns"
            :dataSource="invociDataSource"
            :pagination="invociIpagination"
            :loading="invociLoading"
            @change="handleTableChangeinvoci" style="padding-top: 10px;">

            <span slot="action" slot-scope="text, record">
              <a @click="fileDeteil(record)">附件</a>
              <a-divider type="vertical"/>
              <a @click="handleEdit(record)">详情</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>

      <a-tab-pane tab="回款信息" key="4" style="border-bottom: 1px #e6e5e4 solid;padding-bottom: 14px">
        <div>
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="backId"
            :columns="backColumns"
            :dataSource="backDataSource"
            :pagination="backIpagination"
            :loading="backLoading"
            @change="handleTableChangeback" style="padding-top: 10px;">

            <span slot="action" slot-scope="text, record">
              <a @click="backfileDeteil(record)">附件</a>
              <a-divider type="vertical"/>
              <a @click="handleEditback(record)">详情</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>
    </a-tabs>

    <TenderInfoShow ref="tenderInfoShow" @func="modalFormOk"></TenderInfoShow>

    <project-item-show  ref="projectItemShow" @func="addProjectItem"></project-item-show>

    <invoci-info-file-detail ref="invociInfoFileDetail" @ok="invociInfoShowList"></invoci-info-file-detail>
    <InvociInfoShow ref="invociInfoShow" ></InvociInfoShow>

    <MoneyBackShow ref="moneyBackShow" ></MoneyBackShow>
    <FileDetail ref="fileDetail" @ok="backInfoShowList"></FileDetail>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  import TenderInfoShow from './TenderInfoShow'
  import { httpAction, getAction, deleteAction} from '@/api/manage'
  import {initDictOptions, filterDictText} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import {filterObj} from '@/utils/util'
  import ARow from "ant-design-vue/es/grid/Row";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ACol from "ant-design-vue/es/grid/Col";
  import ProjectItemShow from "./ProjectItemShow";
  import { doMian} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import InvociInfoFileDetail from "./InvociInfoFileDetail";
  import InvociInfoShow from "./InvociInfoShow";
  import MoneyBackShow from "./MoneyBackShow";
  import FileDetail from "./FileDetail";

  export default {
    name: "projectItemModal",
    components: {
      FileDetail,
      ProjectItemShow, ACol, ATextarea, ARow,TenderInfoShow,InvociInfoFileDetail,InvociInfoShow,MoneyBackShow},
    data () {
      return {
        title:"操作",
        visible: false,
        isShow: false,
        model: {},
        //字典数组缓存
        typeDictOptions: [],
        remindTypeOptions: [],
        typePrjDictOptions: [],
        companyNameListA: [],
        companyNameListB: [],
        isOk:true,
        isSearchA: false,
        isSearchB: false,
        isOnFocus: false,
        companyIdA:"",
        companyIdB:"",
        thisMessage:"",
        defaultActiveKey: '1',
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
        avatarElec: "",
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        // 工程点表头
        columns: [
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
            title: '工程编号',
            align: "center",
            dataIndex: 'prjItemNum'
          },
          {
            title: '工程名称',
            align: "center",
            dataIndex: 'prjItemName'
          },
          {
            title: '工程类型',
            align: "center",
            dataIndex: 'prjItemType',
            customRender: (text, record, index) => {
              //字典值替换通用方法
              return filterDictText(this.typePrjDictOptions, text);
            }
          },
          {
            title: '项目名称',
            align: "center",
            dataIndex: 'prjName'
          },
          {
            title: '所属公司',
            align: "center",
            dataIndex: 'companyName'
          },
          {
            title: '负责人',
            align: "center",
            dataIndex: 'personInCharge'
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

        // 发票信息表头
        invociColumns: [
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
            title: '开票日期',
            align: "center",
            dataIndex: 'invociTime'
          },
          {
            title: '开票内容',
            align: "center",
            dataIndex: 'content'
          },
          {
            title: '开票金额',
            align: "center",
            dataIndex: 'totalMoney'
          },
          {
            title: '签收人',
            align: "center",
            dataIndex: 'signatory'
          },
          {
            title: '附件',
            align: "center",
            dataIndex: 'fileRelId',
            customRender: (text) => {
              if(text!=null && text !="" && text != undefined) {
                if(text.indexOf(",") != -1) {
                  var count = text.match(/,/g).length;
                  return parseInt(count);
                }
              }else{
                return 0;
              }
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        //数据集
        invociDataSource: [],
        // 分页参数
        invociIpagination: {
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
        invociIsorter: {
          column: 'createTime',
          order: 'desc',
        },
        invociLoading: false,
        invociSelectedRowKeys: [],
        invociSelectedRows: [],

        // 回款信息表头
        backColumns: [
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
            title: '回款日期',
            align: "center",
            dataIndex: 'backTime'
          },
          {
            title: '回款金额',
            align: "center",
            dataIndex: 'backMoney'
          },
          {
            title: '负责人',
            align: "center",
            dataIndex: 'backPerson'
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'remark'
          },
          {
            title: '附件',
            align: "center",
            dataIndex: 'fileRelId',
            customRender: (text) => {
              if(text!=null && text !="" && text != undefined) {
                if(text.indexOf(",") != -1) {
                  var count = text.match(/,/g).length;
                  return parseInt(count);
                }
              }else{
                return 0;
              }
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        //数据集
        backDataSource: [],
        // 分页参数
        backIpagination: {
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
        backIsorter: {
          column: 'createTime',
          order: 'desc',
        },
        backLoading: false,
        backSelectedRowKeys: [],
        backSelectedRows: [],
        url: {
          add: "/renche/contractInfo/add",
          edit: "/renche/contractInfo/edit",
          searchCompany: "/renche/companyInfo/searchCompany",
          prjItemList: "/renche/contractInfo/contractPrjItemList",
          delprjItem: "/renche/contractInfo/delPrjItem",
          fileUpload: doMian + "/sys/common/upload",
          invociList: "renche/invoci/qryInvociList",
          filelist: "/renche/purchase/fileList",
          backList: "renche/moneyBack/list",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token};
      //初始化字典配置
      this.initDictConfig();
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      },
    },
    methods: {
      initDictConfig() {
        //初始化字典 - 合同类型
        initDictOptions('CONTRACTTYPE').then((res) => {
          if (res.success) {
            this.typeDictOptions = res.result;
          }
        });
        //初始化字典 - 合同提醒周期类型
        initDictOptions('REMINDPERIODTYPE').then((res) => {
          if (res.success) {
            this.remindTypeOptions = res.result;
          }
        });
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
      loadinvociData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.invociIpagination.current = 1;
        }
        getAction(this.url.invociList, {pageNo: this.invociIpagination.current,pageSize: this.invociIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
          if (res.success) {
            this.invociDataSource = res.result.list;
            this.invociIpagination.total = res.result.total;
          }
        })
      },
      loadbackData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.backIpagination.current = 1;
        }
        getAction(this.url.backList, {pageNo: this.backIpagination.current,pageSize: this.backIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
          if (res.success) {
            this.backDataSource = res.result.list;
            this.backIpagination.total = res.result.total;
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
        this.avatarElec = record.elecFileRel == undefined?'':record.elecFileRel;

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;

        if(this.model.contractId != null && this.model.contractId != undefined){
          this.isShow = true;
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model,'contractName','companyNameA','contractNoA','companyNameB','contractNoB','contractType','contractMoney','contractStatus','operatorA','operatorB','remindPeriodType','remindPeriod','prjName','remark'))
            this.form.setFieldsValue({signInTime:this.model.signInTime?moment(this.model.signInTime):null});
            this.form.setFieldsValue({overTime:this.model.overTime?moment(this.model.overTime):null});
            this.form.setFieldsValue({signingTime:this.model.signingTime?moment(this.model.signingTime):null});
          });

          getAction(this.url.prjItemList, {pageNo: this.ipagination.current,pageSize: this.ipagination.pageSize,contractId: this.model.contractId}).then((res) => {
            if (res.success) {
              this.dataSource = res.result.list;
              this.ipagination.total = res.result.total;
            }
          })
          getAction(this.url.invociList, {pageNo: this.invociIpagination.current,pageSize: this.invociIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
            if (res.success) {
              this.invociDataSource = res.result.list;
              this.invociIpagination.total = res.result.total;
            }
          })
          getAction(this.url.backList, {pageNo: this.backIpagination.current,pageSize: this.backIpagination.pageSize,contractId: this.model.contractId}).then((res) => {
            if (res.success) {
              this.backDataSource = res.result.list;
              this.backIpagination.total = res.result.total;
            }
          })
        }
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            if(that.isOk){
              that.confirmLoading = true;
              let httpurl = '';
              let method = '';
              if(!that.model.contractId){
                httpurl+=this.url.add;
                method = 'post';
              }else{
                httpurl+=this.url.edit;
                method = 'put';
              }
              if(that.companyIdA != ''){
                this.model.partyA = that.companyIdA;
              }
              if(that.companyIdB != ''){
                this.model.partyB = that.companyIdB;
              }

              this.model.fileRelId = that.avatar;
              this.model.elecFileRel = that.avatarElec;

              let formData = Object.assign(this.model, values);

              httpAction(httpurl,formData,method).then((res)=>{
                if(res.success){
                  this.model.contractId = res.result.contractId;
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
      handleTableChangeinvoci(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.invociIsorter.column = sorter.field;
          this.invociIsorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadinvociData();
      },
      handleTableChangeback(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.backIsorter.column = sorter.field;
          this.backIsorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadbackData();
      },
      showTender:function (){
        this.$refs.tenderInfoShow.show();
        this.$refs.tenderInfoShow.title = "选择关联招标信息";
      },
      modalFormOk(data) {
        this.model.tenderId = data[0].tenderId;
        this.form.setFieldsValue({prjName:data[0].prjName});
      },
      callback: function(key){
        if(key != 1){
          if(this.model.contractId != undefined && this.model.contractId != ""){
            this.defaultActiveKey = key;
          }else{
            // alert("请先创建合同信息");
            this.$message.warning("请先创建合同信息");
          }
        }else{
          this.defaultActiveKey = key;
        }
      },
      showProjectItem:function (){
        this.$refs.projectItemShow.show(this.model.contractId);
        this.$refs.projectItemShow.title = "选择添加关联工程点信息";
      },
      addProjectItem(data) {
        this.loadData();
      },
      handleDel: function (id) {
        var that = this;
        var contractId = this.model.contractId;
        this.$confirm({
          title: "确认删除",
          content: "是否删除选中数据?",
          onOk: function () {
            deleteAction(that.url.delprjItem, {prjItemId: id, contractId: contractId}).then((res) => {
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
      invociInfoShowList() {
        this.loadinvociData();
      },
      backInfoShowList() {
        this.loadbackData();
      },
      handleChangeElec(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return;
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatarElec = response.message + "," + this.avatarElec;
            console.log(this.avatarElec);
          } else {
            this.$message.warning(response.message);
          }
        }
      },
      handleChange(info) {
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return;
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if (response.success) {
            this.avatar = response.message + "," + this.avatar;
            console.log(this.avatar);
          } else {
            this.$message.warning(response.message);
          }
        }
      },
      fileDeteil:function(record){
        console.log(record);
        this.$refs.invociInfoFileDetail.fileLoad(record);
        this.$refs.invociInfoFileDetail.title = "附件";
      },
      async handleEdit (record) {
        if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
          record.filelist = result.list;
        }
        this.$refs.invociInfoShow.edit(record);
        this.$refs.invociInfoShow.title = "编辑";
      },
      backfileDeteil:function(record){
        console.log(record);
        this.$refs.fileDetail.fileLoad(record);
        this.$refs.fileDetail.title = "附件";
      },
      async handleEditback (record) {
        if(record.fileRelId != null || record.fileRelId != "" || record.fileRelId != undefined) {
          let {result} = await getAction(this.url.filelist, {fileRelId: record.fileRelId});
          record.filelist = result.list;
        }
        this.$refs.moneyBackShow.edit(record);
        this.$refs.moneyBackShow.title = "编辑";
      },
      getCompanyListA: function(val){
        getAction(this.url.searchCompany, {pageNo: "1",pageSize: "5",name: val}).then((res) => {
          if (res.success) {
              this.companyNameListA = res.result.list;
          }
        })
      },
      getCompanyListB: function(val){
        getAction(this.url.searchCompany, {pageNo: "1",pageSize: "5",name: val}).then((res) => {
          if (res.success) {
            this.companyNameListB = res.result.list;
          }
        })
      },
      chooseThisA: function(val){
        this.companyIdA = val;
      },
      chooseThisB: function(val){
        this.companyIdB = val;
      },
    }
  }
</script>

<style scoped>
</style>