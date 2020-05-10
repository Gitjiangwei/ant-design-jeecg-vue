<template>
  <a-modal
    :title="title"
    :width="1050"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-tabs :activeKey="defaultActiveKey" @change="callback" type="card" >
      <a-tab-pane tab="合同信息" key="1" style="padding-bottom: 14px">
        <a-spin :spinning="confirmLoading">
          <a-form :form="form">
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item label="合同名称" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-textarea placeholder="请输入合同名称" v-decorator="['contractName', {rules: [{ required: true, message: '请输入合同名称', }]}]" :autosize="{ minRows: 1, maxRows: 2 }" maxLength="150"/>
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
                  <a-date-picker  v-decorator="['overTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="签收日期" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-date-picker v-decorator="['signInTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="甲方经办人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入甲方经办人" v-decorator="['operatorA', {}]"  maxLength="30" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item label="乙方经办人" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-input placeholder="请输入乙方经办人" v-decorator="['operatorB', {}]"  maxLength="30" />
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
              <a-col :span="16" style="padding-left: 8px;">

              </a-col>
            </a-row>

            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item label="电子版附件" :wrapperCol="wrapperCol" :labelCol="labelCol">
                  <a-upload
                    name="file"
                    :multiple="true"
                    :headers="headers"
                    :customRequest="uploadFileRequest"
                    :file-list="elefileList"
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
                <a-form-item label="扫描件附件" :wrapperCol="wrapperCol" :labelCol="labelCol"><!--  :before-upload="beforeUpload" -->
                  <a-upload
                    name="file"
                    :multiple="true"
                    :headers="headers"
                    :file-list="fileList"
                    :customRequest="uploadFileRequest1"
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

      <a-tab-pane tab="关联工程点" key="2" style="padding-bottom: 14px">
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
              <a @click="prjItemShow(record)">详情</a>
              <a-divider type="vertical"/>
              <a @click="handleDel(record.prjItemId)">删除</a>
            </span>
          </a-table>
        </div>
      </a-tab-pane>

      <a-tab-pane tab="发票信息" key="3" style="padding-bottom: 14px">
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

      <a-tab-pane tab="回款信息" key="4" style="padding-bottom: 14px">
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

    <AddProjectItemRel  ref="addProjectItemRel" @func="addProjectItem"></AddProjectItemRel>
    <ProjectItemShow ref="projectItemShow" ></ProjectItemShow>

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
  import AddProjectItemRel from "./AddProjectItemRel";
  import { doMian} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import InvociInfoFileDetail from "./InvociInfoFileDetail";
  import InvociInfoShow from "../show/InvociInfoShow";
  import MoneyBackShow from "../show/MoneyBackShow";
  import ProjectItemShow from "../show/ProjectItemShow";
  import FileDetail from "./FileDetail";

  export default {
    name: "contractInfoModel",
    components: {
      FileDetail,
      AddProjectItemRel, ACol, ATextarea, ARow,TenderInfoShow,InvociInfoFileDetail,InvociInfoShow,MoneyBackShow,ProjectItemShow},
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
            title: '附件',
            align: "center",
            dataIndex: 'fileCount'
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
        if(record.filelist == undefined){
          record.filelist = [];
        }
        this.isUpload = false;
        this.elecFileRel = record.elecFileRel == undefined?'':record.elecFileRel;
        this.companyIdA = record.partyA;
        this.companyIdB = record.partyB;
        this.elefileList=[];
        this.fileList=[];

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
        if(that.companyIdA == ""){
          this.form.setFieldsValue({companyNameA:""});
        }
        if(that.companyIdB == ""){
          this.form.setFieldsValue({companyNameB:""});
        }
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
                  if(res.result != null){
                    this.model.contractId = res.result.contractId;
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
            this.$message.warning("请先创建合同信息");
          }
        }else{
          this.defaultActiveKey = key;
        }
      },
      showProjectItem:function (){
        this.$refs.addProjectItemRel.show(this.model.contractId);
        this.$refs.addProjectItemRel.title = "选择添加关联工程点信息";
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
      beforeUpload: function (file) {
        return true;
      },

      handleChangeElec(info) {
        if(info.file.status == undefined){
          info.fileList.some((item,i) => {
            if(item.uid == info.file.uid){
              info.fileList.splice(i,1);
            }
          })
        }else{
          if (info.file.status === 'uploading') {
            this.uploadLoading = true
            this.elefileList = [...info.fileList];
            return
          }
          if (info.file.status === 'done') {
            var response = info.file.response;
            this.uploadLoading = false;
            console.log(response);
            if (response.success) {
              this.avatarElec = response.message + "," + this.avatarElec;
              this.elefileList = [...info.fileList];
              let fileItem = info.fileList.slice(-1);
              fileItem[0].id = response.message;
              Vue.set(info.fileList,info.fileList.length-1,fileItem[0])
            } else {
              this.$message.warning(response.message);
            }
            return
          }

          if(info.file.status === 'removed'){
            this.avatarElec = this.avatarElec.replace(info.file.id+',','')
          }
        }
      },


      uploadFileRequest(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const contracName = this.model.contractName;
        var fileName=data.file.name.toString();
        var str1=fileName.substring(0,fileName.lastIndexOf("."));
        var str2=fileName.substring(fileName.lastIndexOf("."), fileName.length);
        const copyFile = new File([data.file], `${contracName}_${nowDate}_${"电子版"}_${str1}_${timeStamp}_${str2}`)
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
      uploadFileRequest1(data){
        const timeStamp = new Date() - 0
        const nowDate = this.getDate();
        const contracName = this.model.contractName;
        var fileName=data.file.name.toString();
        var str1=fileName.substring(0,fileName.lastIndexOf("."));
        var str2=fileName.substring(fileName.lastIndexOf("."), fileName.length);
        const copyFile = new File([data.file], `${contracName}_${nowDate}_${"扫描件"}_${str1}_${timeStamp}_${str2}`)
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
        this.$refs.invociInfoShow.title = "详情";
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
        this.$refs.moneyBackShow.title = "详情";
      },
      getCompanyListA: function(val){
        getAction(this.url.searchCompany, {pageNo: "1",pageSize: "10",name: val}).then((res) => {
          if (res.success) {
            this.companyNameListA = res.result.list;
            if(this.companyNameListA.length == 1){
              if(val == this.companyNameListA[0].companyName){
                this.companyIdA = this.companyNameListA[0].companyId;
              }
            }else{
              this.companyIdA = "";
            }
          }
        })
      },
      getCompanyListB: function(val){
        getAction(this.url.searchCompany, {pageNo: "1",pageSize: "10",name: val}).then((res) => {
          if (res.success) {
            this.companyNameListB = res.result.list;
            if(this.companyNameListB.length == 1){
              if(val == this.companyNameListB[0].companyName){
                this.companyIdB = this.companyNameListB[0].companyId;
              }
            }else{
              this.companyIdB = "";
            }
          }
        })
      },
      chooseThisA: function(val){
        this.companyIdA = val;
      },
      chooseThisB: function(val){
        this.companyIdB = val;
      },
      prjItemShow: function(record){
        this.$refs.projectItemShow.edit(record);
        this.$refs.projectItemShow.title = "详情";
      }
    }
  }
</script>

<style scoped>
</style>