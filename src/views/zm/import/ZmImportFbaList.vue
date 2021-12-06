<template>

  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="12">
            <a-form-item label="FBA ID">
              <j-input placeholder="请输入FBA ID,可模糊查询" v-model="queryParam.fbaid"></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="12">
            <a-form-item label="客户订单号">
              <j-input placeholder="请输入客户订单号" v-model="queryParam.orderid"></j-input>
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

      <div class="table-operator" >

        <a-button @click="handleAdd" type="primary" icon="plus">创建运单</a-button>
        <a-button type="primary" icon="download" @click="handleExportXls('导入fba表')">导出</a-button>
        <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
          <a-button type="primary" icon="import">Excel导入</a-button>
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

      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <template>
          <div>
            <a-tabs default-active-key="0"  type="card" @change="searchQuery"     v-model="queryParam.status">
              <a-tab-pane key="0" tab="已下单"    >
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-button icon="login" @click="showConfirm">按客户数据拣货</a-button>
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button type="danger" @click="change" key='0' > <a-icon type="close" />取消</a-button>
                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="1" tab="已收货" force-render>
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"/>放入出货单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>放入提单
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 配货  </a-button>
                      </a-dropdown>
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="stop">退件</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="vertical-align-top" />转运中
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="issues-close"/> 状态调整  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="2" tab="转运中">
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"/>放入出货单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>放入提单
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 配货  </a-button>
                      </a-dropdown>
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="stop">退件</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="vertical-align-top" />已签收
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="issues-close"/> 状态调整  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="3" tab="已签收">
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"/>放入出货单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>放入提单
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 配货  </a-button>
                      </a-dropdown>
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="stop">退件</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="vertical-align-top" />已签收
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="issues-close"/> 状态调整  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="4" tab="退件">
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="redo">重新发货</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="5" tab="已取消">
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="redo">重新发货</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
              <a-tab-pane key="" tab="全部" >
                <template>
                  <div>
                    <a-button-group size="middle">
                      <a-button icon="printer">打印标签</a-button>
                      <a-button icon="close-circle">拦截/问题件</a-button>
                      <a-button icon="redo">重新发货</a-button>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="edit">
                          <a-menu-item key="1">
                            <a-icon type="edit"/>服务
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="edit"/>供应商服务
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="edit"/>备注
                          </a-menu-item>
                          <a-menu-item key="4" >
                            <a-icon type="edit"/>内部备注
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="edit"/> 批量修改  </a-button>
                      </a-dropdown>
                      <a-dropdown >
                        <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
                          <a-menu-item key="1">
                            <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>运单
                          </a-menu-item>
                          <a-menu-item key="2">
                            <a-icon type="cloud-download"/>货箱
                          </a-menu-item>
                          <a-menu-item key="3" >
                            <a-icon type="cloud-download"/>申报信息
                          </a-menu-item>
                        </a-menu>
                        <a-button><a-icon type="cloud-download"/> 导出  </a-button>
                      </a-dropdown>
                      <a-button icon="plus-circle">入仓</a-button>

                    </a-button-group>
                  </div>
                </template>
              </a-tab-pane>
            </a-tabs>
          </div>
        </template>

      </div>
      <!-- 查询区域-END -->
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="j-table-force-nowrap"
        :scroll="{x:true}"
        :columns="defColumns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">
        <a slot="name" slot-scope="text">{{ text }}</a>
        <a slot="httpurl" slot-scope="text">{{ text }}</a>
        <!--        自定义列-->
        <div slot="filterDropdown">
          <a-card>
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row style="width: 400px">
                <template v-for="(item,index) in columns">
                  <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                    <a-col :span="12"><a-checkbox :value="item.dataIndex"><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type='setting' :style="{ fontSize:'16px',color:  '#108ee9' }" />
        <!--        自定义列-->
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
            <a class="ant-dropdown-link">配货 <a-icon type="down" /></a>
            <a-menu slot="overlay">
               <a-menu-item>
                <a @click="handleDetail(record)">放入出货单</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="handleDetails(record)">放入提单</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
               <a-menu-item>
                <a @click="handleDetail(record)">预览</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="handleDetails(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="changeStatus(record)">拦截/问题件</a>
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

  import { JeecgListMixin } from '../../../mixins/JeecgListMixin'
  // import ZmImportFbaModal from './modules/ZmImportFbaModal#Drawer'
  import ZmImportFbaModal from './modules/ZmImportFbaModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'
  import JInput from '../../../components/jeecg/JInput'
  import Vue from 'vue'

  export default {
    name: "ZmImportFbaList",
    mixins:[JeecgListMixin],
    components: {
      ZmImportFbaModal,
      JInput
    },
    data () {
      return {
        description: '导入FBA表管理页面',
        // 表头
        total: 0,
        showTotal: total => {
          let text = <span>{ ' 共 ' + total + ' 条'}</span>
          return text
        },
        defColumns:[],
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
            title:'运单号',
            align:"center",
            sorter: true,
            dataIndex: 'fbaid',
            scopedSlots: { customRender: 'name' },
            customCell:this.cellClick
          },
          {
            title:'客户订单号',
            sorter: true,
            align:"center",
            dataIndex: 'orderid',
            scopedSlots: { customRender: 'httpurl' },
            customCell:this.cellClick2
          },
          {
            title:'服务',
            align:"center",
            dataIndex: 'serviceId'
          },
          {
            title:'客户公司',
            align:"center",
            dataIndex: 'companySender'
          },
          {
            title:'客户姓名',
            align:"center",
            dataIndex: 'nameSender'
          },
          {
            title:'客户电话',
            align:"center",
            dataIndex: 'telSender'
          },
          {
            title:'客户邮箱',
            align:"center",
            dataIndex: 'emailSender'
          },
          {
            title:'客户地址',
            align:"center",
            dataIndex: 'addressSender'
          },
          {
            title:'客户城市',
            align:"center",
            dataIndex: 'citySender'
          },
          {
            title:'客户省份',
            align:"center",
            dataIndex: 'provinceSender'
          },
          {
            title:'客户邮编',
            align:"center",
            dataIndex: 'countryCode'
          },

          {
            title:'收件人电话',
            align:"center",
            dataIndex: 'countryCodeSender'
          },
          {
            title:'地址库编码',
            align:"center",
            dataIndex: 'code'
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
            title:'收件人姓名',
            align:"center",
            sorter: true,
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
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: {
              filterDropdown: 'filterDropdown',
              filterIcon: 'filterIcon',
              customRender: 'action'},
          }
        ],
        Rowclick: (record, index) => ({
          on: {
            click: () => {
               this.handleDetails(record);
            },
          },
        }),
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
      // this.getSuperFieldList();
      this.initColumns();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {

      //取消订单
      cancelOrder: function(record,){
        this.$http.put('/zmexpress/zmImportFba/change',record).then((response) =>{
          console.log(response);
        })
      },

      //单元格点击事件
      cellClick(record) {
        return {
          style: {
            // 'color': 'blue', //这里将名称变了下色
          },
          on: {
            click: () => { //点击事件，也可以加其他事件
              this.handleDetails(record);
            }
          }
        }
      },
      //单元格点击事件
      cellClick2(record) {
        return {
          style: {
            // 'color': 'blue', //这里将名称变了下色
          },
          on: {
            click: () => { //点击事件，也可以加其他事件
              this.$router.push({path: '/profile/advanced',query:{record: record}})
            }
          }
        }
      },
      //根据tab设置菜单
      settingTab(key){
        console.log('---------------');
        console.log(key);
      },

      //客户数据拣货
      packing: function(){
        console.log(this.selectionRows.length);
          if (this.selectionRows.length==0){
            this.error();
          }
          this.confirm();

      },
      error() {
        this.$error({
          title: '货单未选择',
          content: '请选择一条记录！',
        });
        return;
      },
      confirm() {
        this.$confirm({
          title: '确认fbaid',
          content: this.selectionRows[0].fbaid,
          okText: '确认',
          cancelText: '取消',
          onOk: this.onConfirm()
        });
      },
      showConfirm() {
        if (this.selectionRows.length==0){
          this.error();
        }else {
          this.$confirm({
            title: 'Do you want to modify these items?',
            content: 'When clicked the OK button, this dialog will be closed after 1 second',
            onOk() {
              return new Promise((resolve, reject) => {
                resolve(record);
              }).then(function(value) {
                console.log(value);
              }).catch(() => console.log('Oops errors!'));
            },
            onCancel() {},
          });
        }

      },


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
         fieldList.push({type:'string',value:'status',text:'状态',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>