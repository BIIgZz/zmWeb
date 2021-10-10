<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="提单号">
              <a-input placeholder="请输入提单号" v-model="queryParam.billnum"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="类型">
              <a-input placeholder="请输入类型" v-model="queryParam.type"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="供应商">
                <a-input placeholder="请输入供应商" v-model="queryParam.supplier"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="承运商">
                <a-input placeholder="请输入承运商" v-model="queryParam.arriage"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('提单表')">导出</a-button>
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
        <!--        自定义列-->
        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
          <a-divider type="vertical" />
          <a-popover title="自定义列" trigger="click" placement="leftBottom">
            <template slot="content">
              <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
                <a-row style="width: 400px">
                  <template v-for="(item,index) in columns">
                    <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                        <a-col :span="12"><a-checkbox :value="item.dataIndex"><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox></a-col>
                    </template>
                  </template>
                </a-row>
              </a-checkbox-group>
            </template>
            <a><a-icon type="setting" />设置</a>
          </a-popover>
        </span>
        <!--        自定义列 -->
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

    <zm-billloading-modal ref="modalForm" @ok="modalFormOk"></zm-billloading-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '../../../mixins/JeecgListMixin'
  import ZmBillloadingModal from './modules/ZmBillloadingModal'
  import Vue from 'vue'

  export default {
    name: 'ZmBillloadingList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      ZmBillloadingModal
    },
    data () {
      return {
        description: '提单表管理页面',
        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'提单号',
            align:"center",
            dataIndex: 'billnum'
          },
          {
            title:'类型',
            align:"center",
            dataIndex: 'type'
          },
          {
            title:'供应商',
            align:"center",
            dataIndex: 'supplier'
          },
          {
            title:'承运商',
            align:"center",
            dataIndex: 'arriage'
          },
          {
            title:'清关公司',
            align:"center",
            dataIndex: 'customsClearance'
          },
          {
            title:'路线',
            align:"center",
            dataIndex: 'line'
          },
          {
            title:'用户',
            align:"center",
            dataIndex: 'user'
          },
          {
            title:'箱数',
            align:"center",
            dataIndex: 'boxes'
          },
          {
            title:'合箱数',
            align:"center",
            dataIndex: 'allBoxes'
          },
          {
            title:'实重',
            align:"center",
            dataIndex: 'weight'
          },
          {
            title:'材重',
            align:"center",
            dataIndex: 'materialWeight'
          },
          {
            title:'收费重',
            align:"center",
            dataIndex: 'heavyCharge'
          },
          {
            title:'体积',
            align:"center",
            dataIndex: 'volume'
          },
          {
            title:'盈亏',
            align:"center",
            dataIndex: 'profit'
          },
          {
            title:'备注',
            align:"center",
            dataIndex: 'remark'
          },
          {
            title:'内部备注',
            align:"center",
            dataIndex: 'inRemark'
          },
          {
            title:'出发时间',
            align:"center",
            dataIndex: 'departure',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'到达时间',
            align:"center",
            dataIndex: 'arrive',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'清关时间',
            align:"center",
            dataIndex: 'customsClearanceTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
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
          list: "/zmexpress/zmBillloading/list",
          delete: "/zmexpress/zmBillloading/delete",
          deleteBatch: "/zmexpress/zmBillloading/deleteBatch",
          exportXlsUrl: "/zmexpress/zmBillloading/exportXls",
          importExcelUrl: "zmexpress/zmBillloading/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
     // this.getSuperFieldList();
      this.initColumns();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      //列设置更改事件
      onColSettingsChange (checkedValues) {
        var key = this.$route.name+":colsettings";
        Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
        this.settingColumns = checkedValues;
        const cols = this.columns.filter(item => {
          if(item.key =='rowIndex'|| item.dataIndex=='action'){
            return true
          }
          if (this.settingColumns.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.defColumns =  cols;
      },
      initColumns(){
        //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
        //this.defColumns = colAuthFilter(this.defColumns,'testdemo:');

        var key = this.$route.name+":colsettings";
        let colSettings= Vue.ls.get(key);
        if(colSettings==null||colSettings==undefined){
          let allSettingColumns = [];
          this.columns.forEach(function (item,i,array ) {
            allSettingColumns.push(item.dataIndex);
          })
          this.settingColumns = allSettingColumns;
          this.defColumns = this.columns;
        }else{
          this.settingColumns = colSettings;
          const cols = this.columns.filter(item => {
            if(item.key =='rowIndex'|| item.dataIndex=='action'){
              return true;
            }
            if (colSettings.includes(item.dataIndex)) {
              return true;
            }
            return false;
          })
          this.defColumns =  cols;
        }
      },
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'billnum',text:'提单号',dictCode:''})
        fieldList.push({type:'string',value:'type',text:'类型',dictCode:''})
        fieldList.push({type:'string',value:'supplier',text:'供应商',dictCode:''})
        fieldList.push({type:'string',value:'arriage',text:'承运商',dictCode:''})
        fieldList.push({type:'string',value:'customsClearance',text:'清关公司',dictCode:''})
        fieldList.push({type:'string',value:'line',text:'路线',dictCode:''})
        fieldList.push({type:'string',value:'user',text:'用户',dictCode:''})
        fieldList.push({type:'int',value:'boxes',text:'箱数',dictCode:''})
        fieldList.push({type:'int',value:'allBoxes',text:'合箱数',dictCode:''})
        fieldList.push({type:'string',value:'weight',text:'实重',dictCode:''})
        fieldList.push({type:'string',value:'materialWeight',text:'材重',dictCode:''})
        fieldList.push({type:'string',value:'heavyCharge',text:'收费重',dictCode:''})
        fieldList.push({type:'string',value:'volume',text:'体积',dictCode:''})
        fieldList.push({type:'string',value:'profit',text:'盈亏',dictCode:''})
        fieldList.push({type:'string',value:'remark',text:'备注',dictCode:''})
        fieldList.push({type:'string',value:'inRemark',text:'内部备注',dictCode:''})
        fieldList.push({type:'date',value:'departure',text:'出发时间'})
        fieldList.push({type:'date',value:'arrive',text:'到达时间'})
        fieldList.push({type:'date',value:'customsClearanceTime',text:'清关时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>