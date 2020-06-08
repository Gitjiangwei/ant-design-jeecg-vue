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
    <span style="color: red">需求数量: {{ needNum }} 台</span>
      <div style="float: right;">

        <a-button style="padding-left: 5px;" @click="showTender" type="primary" icon="plus" >新增</a-button>
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

                  <!--<span slot="action" slot-scope="text, record" v-show="isShow1">
                    <a @click="handleDelete(record.outId)" >删除</a>
                  </span>-->
        </a-table>
      </div>



    <purchase-info-stock-show ref="PurchaseInfoStockShow" @ok="addProjectItem"></purchase-info-stock-show>
  </a-modal>
</template>
<script>
  import {httpAction,getAction,deleteAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import PurchaseInfoStockShow from './PurchaseInfoStockShow'
  import {filterObj} from '@/utils/util';

  export default {
    name:"demandPrjModules",
    components:{
      PurchaseInfoStockShow
    },
    data(){
      return{
        isShowFile: false,
        isEdit:false,
        isShow1:true,
        title: "操作",
        visible: false,
        defaultActiveKey: "1",
        dictItemId:"",
        projectId:"",
        needNum:"0",
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
        equipmentName:"",
        equipmentModel:"",
        demandId:"",
        url: {
          add:"renche/demand/saveDemand",
          edit:"renche/demand/updateDemand",
          projectIdList:"/renche/projectItem/projectIdList",
          delOutId: "/renche/outEquip/delOutEquip",
          updateStatus1:"renche/demand/updateStatus1",
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
      getQueryParams() {
        var param = {};
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        if(this.projectId != undefined && this.projectId != null && this.projectId != ""){
          param.projectId = this.projectId;
        }else {
          param.projectId=" ";

        }
        console.log(filterObj(param));
        return filterObj(param);
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
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        getAction(this.url.projectIdList, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.list;
           // alert("length="+this.dataSource.length);
            this.ipagination.total = res.result.total;
          }
        })
      },
      updateStatus(record){
        var that=this;
        deleteAction(that.url.updateStatus1,{demandId:record.demandId,status:"1"}).then((res)=>{
          if(res.success){
            that.$message.success(res.message);
            that.loadData();
          }else {
            that.$message.warning(res.message);
          }
        })
      },

      add() {

        this.edit({});
      },
      edit(record) {
        this.equipmentName=record.equipmentName;
        this.equipmentModel=record.equipmentModel;
        this.demandId=record.demandId;
        this.needNum=record.equipmentNumber;
        this.projectId = record.prjItemId;
        if(record.status=="1"){
          this.isShow1=false;
        }else{
          this.isShow1=true;
        }
        if(this.projectId != undefined && this.projectId != null && this.projectId != ""){
          this.loadData();
        }else {
          this.loadData();
        }


        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;


      },


      showTender:function (){
        var realityNum=this.dataSource.length;
     /* if(this.needNum<=realityNum){
          alert("设备数量已足够，不能再次添加！");
          return;
        }
*/


        if(this.projectId==null || this.projectId == "" || this.projectId == undefined ){
          this.$message.warning("工程點ID為空，不能添加设备");
          return;
        }
        this.$refs.PurchaseInfoStockShow.show(this.projectId,this.equipmentName,this.equipmentModel);
        this.$refs.PurchaseInfoStockShow.title = "选择关联设备信息";
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },

      handleOk() {
        var realityNum=this.dataSource.length;
        if(this.needNum>realityNum){
          alert("设备数量小于需求数量，处理失败！");
          return;
        }else if(this.needNum<realityNum){
          alert("设备数量大于需求数量，不能提交！");
          return;
        }
        const that = this;

       if(realityNum>0){
         deleteAction(that.url.updateStatus1,{demandId:that.demandId,status:"1"}).then((res)=>{
           if(res.success){
            // that.$message.success(res.message);
             that.loadData();
           }else {
             that.$message.warning(res.message);
           }
         })
        }
        that.close();


     /*   // 触发表单验证
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
            formData.isSend = "1";
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
        })*/
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
        var realityNum=this.dataSource.length;
       /* if(this.needNum>realityNum){
          this.$confirm({
            content: "设备数量小于需求数量，确定不再添加吗？?",
            onCancel() {
              return;
            }
          })


        }else */if(this.needNum<realityNum){
          alert("设备数量大于需求数量，请删除多余设备！");
          return;
        }
        this.$emit('ok');
        this.close()
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.isShow = false;
      },

      validateDictOrderNo(rule,value,callback){
        if(value==null||value==""||value==undefined){
          callback("请输入采购数量");
        }
        var reg = /^[0-9]*$/;
        if(!reg.test(value)){
          callback("采购数量只能输入数字");
        }else {
          callback();
        }
      }

    }
  }
</script>