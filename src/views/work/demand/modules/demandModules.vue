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
      <a-form :form="form" >

        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="设备名称">
          <a-input placeholder="请输入设备名称"  maxlength="30"  v-decorator="['equipmentName', {rules: [{ required: true, message: '请输入设备名称', }]}]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="设备型号">
          <a-input placeholder="请输入设备型号" maxlength="30"  v-decorator="['equipmentModel', {rules: [{ required: true, message: '请输入设备型号' }]}]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol"  :wrapperCol="wrapperCol"  label="需求数量">
          <a-input-number v-decorator="['equipmentNumber', {rules: [{ required: true,message: '请输入正确需求数量' }],initialValue: '0'}]" :min="0" :max="999999" :step="1" style="width: 60%;" />
        </a-form-item>

        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="备注">
          <a-textarea  placeholder="请输入备注" maxlength="1000" v-decorator="['remarks', {}]" :autosize="{ minRows: 2, maxRows: 6 }"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
  import {httpAction,getAction} from '@/api/manage'
  import pick from 'lodash.pick'

  export default {
    name:"demandModules",
    components:{},
    data(){
      return{
        isShowFile: false,
        isEdit:"",
        title: "操作",
        visible: false,
        defaultActiveKey: "1",
        dictItemId:"",
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
        //表头
        columns:[
          {
            title: '序号',
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
        uploadLoading: false,
        headers: {},
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add:"renche/demand/saveDemand",
          edit:"renche/demand/updateDemand",
        },
      }
    },
    created() {
    },
    methods:{

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
      add() {
        this.edit({});
      },
      edit(record) {
        this.projectId = record.prjItemId;
        if(this.projectId != undefined && this.projectId != null && this.projectId != ""){
          this.isEdit=false;
        }else {
          this.isEdit=true;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'equipmentName', 'equipmentModel','equipmentNumber','remarks'))
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if (!this.model.demandId) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'post';
            }
            let formData = Object.assign(this.model, values);
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
      callback: function(key){
        if(key != 1){
       /*   if(this.model.prjItemId != undefined && this.model.prjItemId != ""){
            this.defaultActiveKey = key;
          }else{
            this.$message.warning("此设备需求没有工程点，不能关联设备！");
          }*/
          this.defaultActiveKey = key;
        }else{
          this.defaultActiveKey = key;
        }
      },
      handleCancel() {
        this.$emit('ok');
        this.defaultActiveKey = "1";
        this.close()
      },
    }
  }
</script>