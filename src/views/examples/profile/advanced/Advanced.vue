<template>
  <page-layout :title="pageTitle" logo="https://gw.alipayobjects.com/zos/rmsportal/nxkuOJlFJuAUhzlMTCEe.png">

    <detail-list slot="headerContent" size="small" :col="2" class="detail-layout">
      <detail-list-item term="客户">{{ myProp.name}}</detail-list-item>
      <detail-list-item term="目的地">{{ myProp.destination}}</detail-list-item>
      <detail-list-item term="服务类型">{{ myProp.service_dictText}}</detail-list-item>
      <detail-list-item term="目的地">{{ myProp.destination}}</detail-list-item>
      <detail-list-item term="订购产品">{{myProp.enname}}</detail-list-item>
      <detail-list-item term="创建时间">{{ myProp.createTime }}</detail-list-item>
      <detail-list-item term="关联单据"><a>12421</a></detail-list-item>
      <detail-list-item term="创建日期">{{ myProp.createTime }}</detail-list-item>
      <detail-list-item term="备注">{{ myProp.remark }}</detail-list-item>
    </detail-list>
    <a-row slot="extra" class="status-list">
      <a-col :xs="12" :sm="12">
        <div class="text">状态</div>
        <div class="heading" v-if="status == 0">已下单</div>
        <div class="heading" v-else-if="status == 1">已收货</div>
        <div class="heading" v-else-if="status == 2">装运中</div>
        <div class="heading" v-else-if="status == 3">已签收</div>
        <div class="heading" v-else-if="status == 5">退件</div>
        <div class="heading" v-else-if="status == 6">已取消</div>


      </a-col>
      <a-col :xs="12" :sm="12">
        <div class="text">费用</div>
        <div class="heading">¥  568.08 </div>
      </a-col>
    </a-row>
    <!-- actions -->
    <template slot="action">
      <a-button-group style="margin-right: 4px;">
        <a-button>导出运单</a-button>
        <a-button>操作</a-button>
        <a-button><a-icon type="ellipsis"/></a-button>
      </a-button-group>
      <a-button type="primary" >主操作</a-button>
    </template>

    <a-card :bordered="false" title="流程进度">
      <a-steps :direction="isMobile() && 'vertical' || 'horizontal'" :current="1" progressDot>
        <a-step title="创建项目">
        </a-step>
        <a-step title="部门初审">
        </a-step>
        <a-step title="财务复核">
        </a-step>
        <a-step title="完成">
        </a-step>
      </a-steps>
    </a-card>

    <a-card style="margin-top: 24px" :bordered="false" title="基础信息">
      <detail-list>
        <detail-list-item term="收件人">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="发件人">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="身份证">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="报关方式">{{myProp.customsDeclarationMethod_dictText}}</detail-list-item>
        <detail-list-item term="交税方式">{{myProp.taxPaymentMethod_dictText}}</detail-list-item>
        <detail-list-item term="申报币种">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="客户单号">{{myProp.waybillId}}</detail-list-item>
        <detail-list-item term="Amazon Reference ID">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="扩展单号">{{myProp.recipient}}</detail-list-item>
        <detail-list-item term="属性">{{myProp.itemProperties}}</detail-list-item>
      </detail-list>
      <detail-list title="尺寸">
        <detail-list-item term="收费重">---</detail-list-item>
        <detail-list-item term="实重">--</detail-list-item>
        <detail-list-item term="材积重">--</detail-list-item>
        <detail-list-item term="计泡系数">--</detail-list-item>
        <detail-list-item term="体积">--</detail-list-item>
        <detail-list-item term="箱数">--</detail-list-item>
      </detail-list>
      <a-card type="inner" title="多层信息组">
        <detail-list title="组名称" size="small">
          <detail-list-item term="负责人">林东东</detail-list-item>
          <detail-list-item term="角色码">1234567</detail-list-item>
          <detail-list-item term="所属部门">XX公司-YY部</detail-list-item>
          <detail-list-item term="过期时间">2018-08-08</detail-list-item>
          <detail-list-item term="描述">这段描述很长很长很长很长很长很长很长很长很长很长很长很长很长很长...</detail-list-item>
        </detail-list>
        <a-divider style="margin: 16px 0" />
        <detail-list title="组名称" size="small" :col="1">
          <detail-list-item term="学名">	Citrullus lanatus (Thunb.) Matsum. et Nakai一年生蔓生藤本；茎、枝粗壮，具明显的棱。卷须较粗..</detail-list-item>
        </detail-list>
        <a-divider style="margin: 16px 0" />
        <detail-list title="组名称" size="small" :col="2">
          <detail-list-item term="负责人">付小小</detail-list-item>
          <detail-list-item term="角色码">1234567</detail-list-item>
        </detail-list>
      </a-card>

    </a-card>

    <a-card style="margin-top: 24px" :bordered="false" title="用户近半年来电记录">
      <div class="no-data"><a-icon type="frown-o"/>暂无数据</div>
    </a-card>

    <!-- 操作 -->
    <a-card
      style="margin-top: 24px"
      :bordered="false"
      :tabList="tabList"
      :activeTabKey="activeTabKey"
      @tabChange="(key) => {this.activeTabKey = key}"
    >
      <a-table
        v-if="activeTabKey === '1'"
        :columns="operationColumns"
        :dataSource="operation1"
        :pagination="false"
      >
        <template
          slot="status"
          slot-scope="status">
          <a-badge :status="status | statusTypeFilter" :text="status | statusFilter"/>
        </template>
      </a-table>
      <a-table
        v-if="activeTabKey === '2'"
        :columns="operationColumns"
        :dataSource="operation2"
        :pagination="false"
      >
        <template
          slot="status"
          slot-scope="status">
          <a-badge :status="status | statusTypeFilter" :text="status | statusFilter"/>
        </template>
      </a-table>
      <a-table
        v-if="activeTabKey === '3'"
        :columns="operationColumns"
        :dataSource="operation3"
        :pagination="false"
      >
        <template
          slot="status"
          slot-scope="status">
          <a-badge :status="status | statusTypeFilter" :text="status | statusFilter"/>
        </template>
      </a-table>
    </a-card>

  </page-layout>
</template>

<script>
  import { mixinDevice } from '@/utils/mixin.js'
  import PageLayout from '@/components/page/PageLayout'
  import DetailList from '@/components/tools/DetailList'

  const DetailListItem = DetailList.Item

  export default {
    name: "Advanced",
    components: {
      PageLayout,
      DetailList,
      DetailListItem
    },
    mixins: [mixinDevice],
    data () {
      return {
        pageTitle:this.myProp.waybillId+'/'+this.myProp.orderId+'/'+this.myProp.name,
        status:this.myProp.status,
        data:this.myProp,
        tabList: [
          {
            key: '1',
            tab: '品名'
          },
          {
            key: '2',
            tab: '装箱单'
          },
          {
            key: '3',
            tab: '操作日志三'
          }
        ],
        activeTabKey: '1',

        operationColumns: [
          {
            title: '操作类型',
            dataIndex: 'type',
            key: 'type'
          },
          {
            title: '操作人',
            dataIndex: 'name',
            key: 'name'
          },
          {
            title: '执行结果',
            dataIndex: 'status',
            key: 'status',
            scopedSlots: { customRender: 'status' },
          },
          {
            title: '操作时间',
            dataIndex: 'updatedAt',
            key: 'updatedAt'
          },
          {
            title: '备注',
            dataIndex: 'remark',
            key: 'remark'
          }
        ],
        operation1: [
          {
            key: 'op1',
            type: '订购关系生效',
            name: '曲丽丽',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '-'
          },
          {
            key: 'op2',
            type: '财务复审',
            name: '付小小',
            status: 'reject',
            updatedAt: '2017-10-03  19:23:12',
            remark: '不通过原因'
          },
          {
            key: 'op3',
            type: '部门初审',
            name: '周毛毛',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '-'
          },
          {
            key: 'op4',
            type: '提交订单',
            name: '林东东',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '很棒'
          },
          {
            key: 'op5',
            type: '创建订单',
            name: '汗牙牙',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '-'
          }
        ],
        operation2: [
          {
            key: 'op2',
            type: '财务复审',
            name: '付小小',
            status: 'reject',
            updatedAt: '2017-10-03  19:23:12',
            remark: '不通过原因'
          },
          {
            key: 'op3',
            type: '部门初审',
            name: '周毛毛',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '-'
          },
          {
            key: 'op4',
            type: '提交订单',
            name: '林东东',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '很棒'
          }
        ],
        operation3: [
          {
            key: 'op2',
            type: '财务复审',
            name: '付小小',
            status: 'reject',
            updatedAt: '2017-10-03  19:23:12',
            remark: '不通过原因'
          },
          {
            key: 'op3',
            type: '部门初审',
            name: '周毛毛',
            status: 'agree',
            updatedAt: '2017-10-03  19:23:12',
            remark: '-'
          }
        ],
      }
    },
    created() {

    },
    watch:{
      myProp(data){
        this.pageTitle=this.myProp.waybillId+'/'+this.myProp.orderId+'/'+this.myProp.name;
        this.status = this.myProp.status;
      },

    },
    props:{
      myProp:{
        type:Object,
        default:""
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          'agree': '成功',
          'reject': '驳回'
        }
        return statusMap[status]
      },
      statusTypeFilter(type) {
        const statusTypeMap = {
          'agree': 'success',
          'reject': 'error'
        }
        return statusTypeMap[type]
      }
    }
  }
</script>

<style lang="less" scoped>

  .detail-layout {
    margin-left: 44px;
  }
  .text {
    color: rgba(0, 0, 0, .45);
  }

  .heading {
    color: rgba(0, 0, 0, .85);
    font-size: 20px;
  }

  .no-data {
    color: rgba(0, 0, 0, .25);
    text-align: center;
    line-height: 64px;
    font-size: 16px;

    i {
      font-size: 24px;
      margin-right: 16px;
      position: relative;
      top: 3px;
    }
  }

  .mobile {
    .detail-layout {
      margin-left: unset;
    }
    .text {

    }
    .status-list {
      text-align: left;
    }
  }
</style>