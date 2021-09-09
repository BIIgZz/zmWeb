<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="服务名称">
              <a-input placeholder="请输入服务名称" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="服务代码">
              <a-input placeholder="请输入服务代码" v-model="queryParam.serviceCode"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="状态">
                <j-dict-select-tag placeholder="请选择状态" v-model="queryParam.status" dictCode="service_status"/>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
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
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('zm_service')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
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

    <zm-service-modal ref="modalForm" @ok="modalFormOk"></zm-service-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ZmServiceModal from './modules/ZmServiceModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: 'ZmServiceList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      ZmServiceModal
    },
    data () {
      return {
        description: 'zm_service管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'服务名称',
            align:"center",
            dataIndex: 'name'
          },
          {
            title:'服务代码',
            align:"center",
            dataIndex: 'serviceCode'
          },
          {
            title:'服务分类',
            align:"center",
            dataIndex: 'serviceSort_dictText'
          },
          {
            title:'计费方式',
            align:"center",
            dataIndex: 'billingPlan_dictText'
          },
          {
            title:'计重方式',
            align:"center",
            dataIndex: 'weighingMethod_dictText'
          },
          {
            title:'计泡系数',
            align:"center",
            dataIndex: 'bubble'
          },
          {
            title:'分泡比例',
            align:"center",
            dataIndex: 'bubbleSplittingRatio'
          },
          {
            title:'创建人',
            align:"center",
            dataIndex: 'createBy'
          },
          {
            title:'创建日期',
            align:"center",
            dataIndex: 'createTime'
          },
          {
            title:'更新人',
            align:"center",
            dataIndex: 'updateBy'
          },
          {
            title:'update_time',
            align:"center",
            dataIndex: 'updateTime'
          },
          {
            title:'所属部门',
            align:"center",
            dataIndex: 'sysOrgCode'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/zmexpress/zmService/list",
          delete: "/zmexpress/zmService/delete",
          deleteBatch: "/zmexpress/zmService/deleteBatch",
          exportXlsUrl: "/zmexpress/zmService/exportXls",
          importExcelUrl: "zmexpress/zmService/importExcel",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'name',text:'服务名称',dictCode:''})
        fieldList.push({type:'string',value:'serviceCode',text:'服务代码',dictCode:''})
        fieldList.push({type:'string',value:'serviceSort',text:'服务分类',dictCode:'service_sort'})
        fieldList.push({type:'string',value:'billingPlan',text:'计费方式',dictCode:'bill'})
        fieldList.push({type:'string',value:'weighingMethod',text:'计重方式',dictCode:'weight'})
        fieldList.push({type:'string',value:'bubble',text:'计泡系数',dictCode:''})
        fieldList.push({type:'string',value:'bubbleSplittingRatio',text:'分泡比例',dictCode:''})
        fieldList.push({type:'string',value:'salesCommissionBase',text:'销售提成基数',dictCode:''})
        fieldList.push({type:'string',value:'salesCommission',text:'销售提成比例',dictCode:''})
        fieldList.push({type:'string',value:'salesCommissionUnitPrice',text:'销售提成单价',dictCode:''})
        fieldList.push({type:'string',value:'numberRules',text:'运单号规则',dictCode:''})
        fieldList.push({type:'string',value:'tagCode',text:'标签代码',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'service_status'})
        fieldList.push({type:'string',value:'sender',text:'发件人',dictCode:''})
        fieldList.push({type:'string',value:'customsDeclarationMethod',text:'支持报关方式',dictCode:'customs_eclaration'})
        fieldList.push({type:'string',value:'customsLearanceMethod',text:'支持清关方式',dictCode:'customs_clearance'})
        fieldList.push({type:'list_multi',value:'taxPaymentMethod',text:'支持交税方式',dictTable:'', dictText:'', dictCode:'tax_payment'})
        fieldList.push({type:'string',value:'deliveryTerms',text:'支持交货条款',dictCode:'delivery_terms'})
        fieldList.push({type:'string',value:'electric',text:'带电标识',dictCode:''})
        fieldList.push({type:'string',value:'boxWeight',text:'最低箱收费重',dictCode:''})
        fieldList.push({type:'string',value:'country',text:'到达国家',dictCode:''})
        fieldList.push({type:'list_multi',value:'reportingCurrency',text:'申报币种',dictTable:'', dictText:'', dictCode:'currency'})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'string',value:'createTime',text:'创建日期',dictCode:''})
        fieldList.push({type:'string',value:'updateBy',text:'更新人',dictCode:''})
        fieldList.push({type:'string',value:'updateTime',text:'update_time',dictCode:''})
        fieldList.push({type:'string',value:'sysOrgCode',text:'所属部门',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>