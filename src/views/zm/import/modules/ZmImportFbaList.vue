<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="FBA ID">
              <a-input placeholder="请输入FBA ID" v-model="queryParam.fbaid"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="客户订单号">
              <a-input placeholder="请输入客户订单号" v-model="queryParam.orderid"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="地址库编码">
                <a-input placeholder="请输入地址库编码" v-model="queryParam.code"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('导入FBA表')">导出</a-button>
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
        bordered
        rowKey="id"
        class="j-table-force-nowrap"
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
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

    <zm-import-fba-modal ref="modalForm" @ok="modalFormOk"/>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ZmImportFbaModal from './modules/ZmImportFbaModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'

  export default {
    name: "ZmImportFbaList",
    mixins:[JeecgListMixin],
    components: {
      ZmImportFbaModal
    },
    data () {
      return {
        description: '导入FBA表管理页面',
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
            title:'FBA ID',
            align:"center",
            sorter: true,
            dataIndex: 'fbaid'
          },
          {
            title:'客户订单号',
            align:"center",
            dataIndex: 'orderid'
          },
          {
            title:'服务',
            align:"center",
            dataIndex: 'serviceId'
          },
          {
            title:'地址库编码',
            align:"center",
            dataIndex: 'code'
          },
          {
            title:'收件人姓名',
            align:"center",
            dataIndex: 'name'
          },
          {
            title:'收件人公司',
            align:"center",
            dataIndex: 'company'
          },
          {
            title:'收件人地址',
            align:"center",
            dataIndex: 'address'
          },
          {
            title:'收件人城市',
            align:"center",
            dataIndex: 'city'
          },
          {
            title:'收件人省份',
            align:"center",
            dataIndex: 'province'
          },
          {
            title:'收件人邮编',
            align:"center",
            dataIndex: 'postcode'
          },
          {
            title:'收件人国家代码',
            align:"center",
            dataIndex: 'countryCode'
          },
          {
            title:'收件人电话',
            align:"center",
            dataIndex: 'tel'
          },
          {
            title:'收件人邮箱',
            align:"center",
            dataIndex: 'email'
          },
          {
            title:'总箱数',
            align:"center",
            dataIndex: 'caseNumber'
          },
          {
            title:'是否带电',
            align:"center",
            dataIndex: 'electrical_dictText'
          },
          {
            title:'是否带磁性',
            align:"center",
            dataIndex: 'magnetic_dictText'
          },
          {
            title:'是否是液体',
            align:"center",
            dataIndex: 'liquid_dictText'
          },
          {
            title:'是否粉末',
            align:"center",
            dataIndex: 'powder_dictText'
          },
          {
            title:'是否是危险品',
            align:"center",
            dataIndex: 'dangerous_dictText'
          },
          {
            title:'报关方式',
            align:"center",
            dataIndex: 'customsEclaration_dictText'
          },
          {
            title:'清关方式',
            align:"center",
            dataIndex: 'customsClearance_dictText'
          },
          {
            title:'交税方式',
            align:"center",
            dataIndex: 'taxPayment_dictText'
          },
          {
            title:'交货条款',
            align:"center",
            dataIndex: 'deliveryTerm'
          },
          {
            title:'VAT号',
            align:"center",
            dataIndex: 'vat'
          },
          {
            title:'参考号1',
            align:"center",
            dataIndex: 'referenceNumber1'
          },
          {
            title:'参考号2',
            align:"center",
            dataIndex: 'referenceNumber2'
          },
          {
            title:'备注',
            align:"center",
            dataIndex: 'note'
          },
          {
            title:'发件人公司',
            align:"center",
            dataIndex: 'companySender'
          },
          {
            title:'发件人姓名',
            align:"center",
            dataIndex: 'nameSender'
          },
          {
            title:'发件人地址',
            align:"center",
            dataIndex: 'addressSender'
          },
          {
            title:'发件人城市',
            align:"center",
            dataIndex: 'citySender'
          },
          {
            title:'发件人省份',
            align:"center",
            dataIndex: 'provinceSender'
          },
          {
            title:'发件人邮编',
            align:"center",
            dataIndex: 'postcodeSender'
          },
          {
            title:'发件人国家代码',
            align:"center",
            dataIndex: 'countryCodeSender'
          },
          {
            title:'发件人电话',
            align:"center",
            dataIndex: 'telSender'
          },
          {
            title:'发件人邮箱',
            align:"center",
            dataIndex: 'emailSender'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/zmexpress/zmImportFba/list",
          delete: "/zmexpress/zmImportFba/delete",
          deleteBatch: "/zmexpress/zmImportFba/deleteBatch",
          exportXlsUrl: "/zmexpress/zmImportFba/exportXls",
          importExcelUrl: "zmexpress/zmImportFba/importExcel",
          
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
      }
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
         fieldList.push({type:'string',value:'fbaid',text:'FBA ID',dictCode:''})
         fieldList.push({type:'string',value:'orderid',text:'客户订单号',dictCode:''})
         fieldList.push({type:'string',value:'serviceId',text:'服务',dictCode:''})
         fieldList.push({type:'string',value:'code',text:'地址库编码',dictCode:''})
         fieldList.push({type:'string',value:'name',text:'收件人姓名',dictCode:''})
         fieldList.push({type:'string',value:'company',text:'收件人公司',dictCode:''})
         fieldList.push({type:'string',value:'address',text:'收件人地址',dictCode:''})
         fieldList.push({type:'string',value:'city',text:'收件人城市',dictCode:''})
         fieldList.push({type:'string',value:'province',text:'收件人省份',dictCode:''})
         fieldList.push({type:'string',value:'postcode',text:'收件人邮编',dictCode:''})
         fieldList.push({type:'string',value:'countryCode',text:'收件人国家代码',dictCode:''})
         fieldList.push({type:'string',value:'tel',text:'收件人电话',dictCode:''})
         fieldList.push({type:'string',value:'email',text:'收件人邮箱',dictCode:''})
         fieldList.push({type:'int',value:'caseNumber',text:'总箱数',dictCode:''})
         fieldList.push({type:'int',value:'electrical',text:'是否带电',dictCode:'judgment'})
         fieldList.push({type:'int',value:'magnetic',text:'是否带磁性',dictCode:'judgment'})
         fieldList.push({type:'int',value:'liquid',text:'是否是液体',dictCode:'judgment'})
         fieldList.push({type:'int',value:'powder',text:'是否粉末',dictCode:'judgment'})
         fieldList.push({type:'int',value:'dangerous',text:'是否是危险品',dictCode:'judgment'})
         fieldList.push({type:'int',value:'customsEclaration',text:'报关方式',dictCode:'customs_eclaration'})
         fieldList.push({type:'int',value:'customsClearance',text:'清关方式',dictCode:'customs_clearance'})
         fieldList.push({type:'int',value:'taxPayment',text:'交税方式',dictCode:'tax_payment'})
         fieldList.push({type:'string',value:'deliveryTerm',text:'交货条款',dictCode:''})
         fieldList.push({type:'string',value:'vat',text:'VAT号',dictCode:''})
         fieldList.push({type:'string',value:'referenceNumber1',text:'参考号1',dictCode:''})
         fieldList.push({type:'string',value:'referenceNumber2',text:'参考号2',dictCode:''})
         fieldList.push({type:'string',value:'note',text:'备注',dictCode:''})
         fieldList.push({type:'string',value:'companySender',text:'发件人公司',dictCode:''})
         fieldList.push({type:'string',value:'nameSender',text:'发件人姓名',dictCode:''})
         fieldList.push({type:'string',value:'addressSender',text:'发件人地址',dictCode:''})
         fieldList.push({type:'string',value:'citySender',text:'发件人城市',dictCode:''})
         fieldList.push({type:'string',value:'provinceSender',text:'发件人省份',dictCode:''})
         fieldList.push({type:'string',value:'postcodeSender',text:'发件人邮编',dictCode:''})
         fieldList.push({type:'string',value:'countryCodeSender',text:'发件人国家代码',dictCode:''})
         fieldList.push({type:'string',value:'telSender',text:'发件人电话',dictCode:''})
         fieldList.push({type:'string',value:'emailSender',text:'发件人邮箱',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>