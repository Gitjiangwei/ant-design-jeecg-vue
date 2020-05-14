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
                  <a-textarea readonly v-decorator="['prjItemName', {}]" :autosize="{ minRows: 1, maxRows: 2 }" class="read_tex"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目名称" >
                  <a-textarea readonly v-decorator="['prjName', {}]" :autosize="{ minRows: 1, maxRows: 2 }" class="read_tex"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程编号" >
                  <a-input disabled v-decorator="['prjItemNum', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程类型" >
                  <a-select v-decorator="['prjItemType', {}]" disabled>
                    <a-select-option v-for="item in typeDictOptions" :key="item.value" :value="item.value">{{item.text}}</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="所属公司" >
                  <a-input v-decorator="['companyName', {}]" disabled/>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="负责人" >
                  <a-input disabled v-decorator="['personInCharge', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="联系电话" >
                  <a-input disabled v-decorator="['personTel', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程状态" >
                  <a-select v-decorator="['prjItemStatus', {}]" disabled>
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
                  <a-select v-decorator="['buildType', {}]" disabled>
                    <a-select-option value="0">迁移</a-select-option>
                    <a-select-option value="1">新建</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="拥有方式" >
                  <a-select v-decorator="['haveWay', {}]" disabled>
                    <a-select-option value="0">租赁</a-select-option>
                    <a-select-option value="1">购买</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="进场时间" >
                  <a-date-picker disabled v-decorator="[ 'entryTime', {}]" />
                </a-form-item>
              </a-col>
              <a-col :span="12" style="padding-left: 0px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="完成时间" >
                  <a-date-picker disabled  v-decorator="[ 'finishTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="12" style="padding-left: 40px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="要求部署时间" >
                  <a-date-picker disabled v-decorator="['requireDeployTime', {}]" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程进度" >
                  <a-textarea readonly v-decorator="['progressOfItem', {}]" :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex"/>
                </a-form-item>
              </a-col>
            </a-row>
            <a-row :gutter="24">
              <a-col :span="16" style="padding-left: 8px;">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="工程地址" >
                  <a-textarea  readonly v-decorator="['prjItemPlace', {}]" :autosize="{ minRows: 2, maxRows: 6 }" class="read_tex"  />
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
        </a-spin>
      </a-tab-pane>
      <a-tab-pane tab="关联设备" key="2" style="padding-bottom: 14px">

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

          </a-table>
        </div>
      </a-tab-pane>
    </a-tabs>

    <div slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </div>
  </a-modal>
</template>

<script>
  import { getAction } from '@/api/manage'
  import {initDictOptions} from '@/components/dict/RencheDictSelectUtil'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ACol from "ant-design-vue/es/grid/Col";
  import ATextarea from "ant-design-vue/es/input/TextArea";
  import ARow from "ant-design-vue/es/grid/Row";
  import {filterObj} from '@/utils/util';

  export default {
    name: "projectItemModal",
    components: {ACol, ATextarea, ARow},
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        isOk: true,
        projectId:"",
        companyId:"",
        thisMessage: "",
        defaultActiveKey: "1",
        dateFormat: 'YYYY-MM-DD',
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
        validatorRules:{
        },
        companyNameList: [],
        url: {
          projectIdList:"/renche/projectItem/projectIdList",
        },
      }
    },
    created () {
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
      edit (record) {
        this.form.resetFields();
        this.dataSource = [];
        this.projectId = record.prjItemId;
        if(this.projectId != undefined && this.projectId != null && this.projectId != ""){
          this.loadData();
        }
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'prjItemName','prjItemNum','prjItemType','prjName','prjItemStatus','companyName','personInCharge','personTel','progressOfItem','prjItemPlace','buildType','haveWay'))
          this.form.setFieldsValue({entryTime:this.model.entryTime?moment(this.model.entryTime):null});
          this.form.setFieldsValue({finishTime:this.model.finishTime?moment(this.model.finishTime):null});
          this.form.setFieldsValue({requireDeployTime:this.model.requireDeployTime?moment(this.model.requireDeployTime):null});
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
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
    }
  }
</script>

<style scoped>
  .read_tex{
    background-color: #e6e5e56b;
    color: #8b89898a;
  }
</style>