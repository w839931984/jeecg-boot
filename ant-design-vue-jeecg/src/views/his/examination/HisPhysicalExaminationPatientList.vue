<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="体检卡号">
              <a-input placeholder="请输入体检卡号" v-model="queryParam.cardNumber" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="体检人姓名">
              <a-input placeholder="请输入体检人姓名" v-model="queryParam.patientName"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="性别">
                <a-select v-model="queryParam.patientSex" placeholder="请选择性别查询">
                  <a-select-option value="">请选择性别查询</a-select-option>
                  <a-select-option value="0">未知</a-select-option>
                  <a-select-option value="1">男性</a-select-option>
                  <a-select-option value="2">女性</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="身份证号">
                <a-input placeholder="请输入身份证号" v-model="queryParam.idNumber"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('体检人员管理')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
          <a-menu-item key="2" @click="batchFrozen('1')">
            <a-icon type="lock"/>
            冻结
          </a-menu-item>
          <a-menu-item key="3" @click="batchFrozen('0')">
            <a-icon type="unlock"/>
            解冻
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
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
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleEdit(record)">编辑</a>
              </a-menu-item>
              <a-menu-item v-if="record.status==0">
                <a-popconfirm title="确定冻结吗?" @confirm="() => handleFrozen(record.id,1)">
                  <a>冻结</a>
                </a-popconfirm>
              </a-menu-item>

              <a-menu-item v-if="record.status==1">
                <a-popconfirm title="确定解冻吗?" @confirm="() => handleFrozen(record.id,0)">
                  <a>解冻</a>
                </a-popconfirm>
              </a-menu-item>
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

    <!-- 表单区域 -->
    <hisPhysicalExaminationPatient-modal ref="modalForm" @ok="modalFormOk"></hisPhysicalExaminationPatient-modal>
  </a-card>
</template>

<script>
  import HisPhysicalExaminationPatientModal from './modules/HisPhysicalExaminationPatientModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import {frozenBatch} from '@/api/api'

  export default {
    name: "HisPhysicalExaminationPatientList",
    mixins: [JeecgListMixin],
    components: {
      HisPhysicalExaminationPatientModal
    },
    data() {
      return {
        /* 排序参数 */
        isorter:{
          column: 'registerTime',
          order: 'desc',
        },
        description: '体检人员管理管理页面',
        // 表头
        columns: [
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          },
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
            title: '体检卡号',
            align: "center",
            dataIndex: 'cardNumber'
          },
          {
            title: '体检人姓名',
            align: "center",
            dataIndex: 'patientName'
          },
          {
            title: '性别',
            align: "center",
            dataIndex: 'patientSex_dictText'
          },
          {
            title: '生日',
            align: "center",
            dataIndex: 'patientBirthday'
          },
          {
            title: '年龄',
            align: "center",
            dataIndex: 'patientAge'
          },
          {
            title: '身份证号',
            align: "center",
            dataIndex: 'idNumber'
          },
          {
            title: '民族',
            align: "center",
            dataIndex: 'nation_dictText'
          },
          {
            title: '电子邮件',
            align: "center",
            dataIndex: 'email'
          },
          {
            title: '电话',
            align: "center",
            dataIndex: 'phone'
          },
          {
            title: '籍贯',
            align: "center",
            dataIndex: 'nativePlace'
          },
          {
            title: '婚姻状况',
            align: "center",
            dataIndex: 'maritalStatus_dictText'
          },
          {
            title: '照片',
            align: "center",
            dataIndex: 'photo'
          },
          {
            title: '邮编',
            align: "center",
            dataIndex: 'postcode'
          },
          {
            title: '联系地址',
            align: "center",
            dataIndex: 'contactAddress'
          },
          {
            title: '单位编码',
            align: "center",
            dataIndex: 'companyCode'
          },
          {
            title: '部门编码',
            align: "center",
            dataIndex: 'departCode'
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'remarks'
          },
          {
            title: '登记人',
            align: "center",
            dataIndex: 'registerBy'
          },
          {
            title: '登记部门',
            align: "center",
            dataIndex: 'registerDepart'
          },
          {
            title: '登记时间',
            align: "center",
            dataIndex: 'registerTime'
          },
          {
            title: '更新部门',
            align: "center",
            dataIndex: 'updateDepart'
          },
          {
            title: '使用状态',
            align: "center",
            dataIndex: 'status_dictText'
          }
          // {
          //   title: '删除状态(0-正常,1-已删除)',
          //   align: "center",
          //   dataIndex: 'delFlag'
          // },
        ],
        url: {
          list: "/his/examination/hisPhysicalExaminationPatient/list",
          delete: "/his/examination/hisPhysicalExaminationPatient/delete",
          deleteBatch: "/his/examination/hisPhysicalExaminationPatient/deleteBatch",
          frozenBatch: "/his/examination/hisPhysicalExaminationPatient/frozenBatch",
          exportXlsUrl: "/his/examination/hisPhysicalExaminationPatient/exportXls",
          importExcelUrl: "/his/examination/hisPhysicalExaminationPatient/importExcel",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {

    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>