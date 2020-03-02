<template>
    <a-card :bordered="false" >
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">

            <a-col :span="6">
              <a-form-item label="物品名称">
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

<!--        <a-dropdown v-if="selectedRowKeys.length > 0">-->
<!--          <a-menu slot="overlay">-->
<!--            <a-menu-item key="1" @click="batchDel">-->
<!--              <a-icon type="delete"/>-->
<!--              删除-->
<!--            </a-menu-item>-->
<!--          </a-menu>-->
<!--          <a-button style="margin-left: 8px"> 批量操作-->
<!--            <a-icon type="down"/>-->
<!--          </a-button>-->
<!--        </a-dropdown>-->
      </div>

      <!-- table区域-begin -->
      <div>
        <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
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


    </a-card>

</template>

<script>
    import ARow from "ant-design-vue/es/grid/Row";
    export default {
      name: "purchaseInfo",
      components: {ARow},
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
              title: '姓名',
              align: "center",
              dataIndex: 'name'
            },
            {
              title: '关键词',
              align: "center",
              dataIndex: 'keyWord'
            },
            {
              title: '打卡时间',
              align: "center",
              dataIndex: 'punchTime'
            },
            {
              title: '性别',
              align: "center",
              dataIndex: 'sex',
              customRender: (text, record, index) => {
                //字典值替换通用方法
                return filterDictText(this.sexDictOptions, text);
              }
            },
            {
              title: '年龄',
              align: "center",
              dataIndex: 'age'
            },
            {
              title: '生日',
              align: "center",
              dataIndex: 'birthday'
            },
            {
              title: '邮箱',
              align: "center",
              dataIndex: 'email'
            },
            {
              title: '个人简介',
              align: "center",
              dataIndex: 'content'
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
            delete: "/test/jeecgDemo/delete",
            deleteBatch: "/test/jeecgDemo/deleteBatch",
          },
        }
      },
      methods: {
        handleAdd: function () {
          // this.$refs.jeecgDemoModal.add();
          // this.$refs.jeecgDemoModal.title = "新增";
        },
        searchReset() {
          var that = this;
          that.queryParam = {}
          that.loadData(1);
        }
      }
    }

</script>

<style scoped>

</style>