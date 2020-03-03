<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">

            <a-col :span="6">
              <a-form-item label="物品名称" >
                <a-input placeholder="请输入物品名称" v-model="queryParam.purchaseName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="采购人">
                <a-input placeholder="请输入采购人" v-model="queryParam.purchaser"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="是否到货">
                <a-select v-model="queryParam.isArrival">
                  <a-select-option :key="1">是</a-select-option>
                  <a-select-option :key="0">否</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="到货时间">
                <a-date-picker></a-date-picker>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row>
            <a-col :span="6"  >
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <!--<a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>-->
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <!-- 操作按钮区域 -->
      <div class="table-operator">
        <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>

<!--        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1" @click="batchDel">
              <a-icon type="delete"/>
              删除
            </a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px"> 批量操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>-->
      </div>
      <!-- table区域-begin -->
      <div>
        <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;margin-top: 15px;">
          <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
          selectedRowKeys.length }}</a>项
          <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        </div>

        <a-table
          ref="table"
          size="middle"
          bordered
          rowKey="id"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="ipagination"
          :loading="loading"
          :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
          @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

        </a-table>
      </div>
      <!-- table区域-end -->
      <prochase-info-mode ref="prochaseInfoMode" @ok="modalFormOk"></prochase-info-mode>

    </a-card>

</template>

<script>
    import ARow from "ant-design-vue/es/grid/Row";
    import prochaseInfoMode from "./modules/pruchaseInfoMode";
    import {deleteAction, getAction, postAction} from '@/api/manage'

    export default {
      name: "purchaseInfo",
      components: {
        ARow,
        prochaseInfoMode
      },
      data() {
        return{
          description: '采购管理页面',

          // 查询条件
          queryParam: {},
          // 表头
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
              title: '物品名称',
              align: "center",
              dataIndex: 'purchaseItem'
            },
            {
              title: '设备型号',
              align: "center",
              dataIndex: 'itemModel'
            },
            {
              title: '单价',
              align: "center",
              dataIndex: 'price'
            },
            {
              title: '数量',
              align: "center",
              dataIndex: 'quantity',
/*              customRender: (text, record, index) => {
                //字典值替换通用方法
                return filterDictText(this.sexDictOptions, text);
              }*/
            },
            {
              title: '总价',
              align: "center",
              dataIndex: 'totalPrice',
            },
            {
              title: '采购人员',
              align: "center",
              dataIndex: 'purchaser'
            },
            {
              title: '采购时间',
              align: "center",
              dataIndex: 'purchase_time'
            },
            {
              title: '采购来源',
              align: "center",
              dataIndex: 'whichCompany'
            },
            {
              title: '到货日期',
              align: "center",
              dataIndex: 'arrivalTime'
            },
            {
              title: '是否到货',
              align: "center",
              dataIndex: 'isarrival'
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
            total: 0
          },
          isorter: {
            column: 'createTime',
            order: 'desc',
          },
          loading: false,
          selectedRowKeys: [],
          selectedRows: [],
          url: {
            list: "/test/jeecgDemo/list",
/*            delete: "/test/jeecgDemo/delete",
            deleteBatch: "/test/jeecgDemo/deleteBatch",*/
          },
        }
      },
      created() {
        this.loadData();
        //初始化字典配置
        this.initDictConfig();
      },
      methods: {
      loadData(arg) {
          //加载数据 若传入参数1则加载第一页的内容
          if (arg === 1) {
            this.ipagination.current = 1;
          }
          var params = this.getQueryParams();//查询条件
          getAction(this.url.list, params).then((res) => {
            if (res.success) {
              this.dataSource = res.result.records;
              this.ipagination.total = res.result.total;
            }
          })
        },
        handleAdd: function () {
           this.$refs.prochaseInfoMode.add();
           this.$refs.prochaseInfoMode.title = "新增";
        },
      modalFormOk() {
          // 新增/修改 成功时，重载列表
          this.loadData();
        },
        searchReset() {
          var that = this;
          that.queryParam = {}
          that.loadData(1);
        },
        getQueryParams() {
          var param = Object.assign({}, this.queryParam, this.isorter);
          param.field = this.getQueryField();
          param.pageNo = this.ipagination.current;
          param.pageSize = this.ipagination.pageSize;
          return filterObj(param);
        },
        getQueryField() {
          //TODO 字段权限控制
          var str = "id,";
          this.columns.forEach(function (value, index) {
            str += "," + value.dataIndex;
          });
          return str;
        },
      }
    }

</script>

<style scoped>

</style>