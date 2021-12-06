<template>
  <page-layout :title="title">
    <a-card :bordered="false">

      <detail-list title="订单详情">
        <detail-list-item term="FBI ID">{{fbaDtail.fbaid}}</detail-list-item>
        <detail-list-item term="客户订单号">{{fbaDtail.orderid}}</detail-list-item>
        <detail-list-item term="服务">{{ fbaDtail.serviceId}}</detail-list-item>
        <detail-list-item term="地址库编码">{{ fbaDtail.code }}</detail-list-item>
      </detail-list>

      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="发货客户信息">
        <detail-list-item term="客户姓名">{{ fbaDtail.nameSender }}</detail-list-item>
        <detail-list-item term="联系电话">{{ fbaDtail.telSender}}</detail-list-item>
        <detail-list-item term="联系邮箱">{{ fbaDtail.emailSender}}</detail-list-item>
        <detail-list-item term="邮编">{{ fbaDtail.postcodeSender}}</detail-list-item>
        <detail-list-item term="国家代码(二字代码)">{{ fbaDtail.countryCodeSender}}</detail-list-item>
        <detail-list-item term="省份/州*(二字代码)">{{ fbaDtail.provinceSender}}</detail-list-item>
        <detail-list-item term="城市">	{{ fbaDtail.citySender}}</detail-list-item>
        <detail-list-item term="收件地址">	{{ fbaDtail.addressSender}}</detail-list-item>
        <detail-list-item term="创建人">	{{ dataList[0].createBy}}</detail-list-item>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="收件人信息">
        <detail-list-item term="用户姓名">{{ fbaDtail.name }}</detail-list-item>
        <detail-list-item term="联系电话">{{ fbaDtail.tel}}</detail-list-item>
        <detail-list-item term="联系邮箱">{{ fbaDtail.email}}</detail-list-item>
        <detail-list-item term="邮编">{{ fbaDtail.postcode}}</detail-list-item>
        <detail-list-item term="国家代码(二字代码)">{{ fbaDtail.countryCode}}</detail-list-item>
        <detail-list-item term="省份/州*(二字代码)">{{ fbaDtail.province}}</detail-list-item>
        <detail-list-item term="城市">	{{ fbaDtail.city}}</detail-list-item>
        <detail-list-item term="收件地址">	{{ fbaDtail.address}}</detail-list-item>
        <detail-list-item term="PO Numbe">	123456</detail-list-item>
        <detail-list-item term="箱数">	12</detail-list-item>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>

      <detail-list title="订单特性">
        <detail-list-item term="带电">{{ fbaDtail.electrical_dictText }}</detail-list-item>
        <detail-list-item term="带磁">{{ fbaDtail.magnetic_dictText}}</detail-list-item>
        <detail-list-item term="液体">{{ fbaDtail.liquid_dictText}}</detail-list-item>
        <detail-list-item term="粉末">{{ fbaDtail.powder_dictText}}</detail-list-item>
        <detail-list-item term="危险品">{{ fbaDtail.dangerous_dictText}}</detail-list-item>
        <detail-list-item term="报关方式">{{ fbaDtail.customsEclaration_dictText}}</detail-list-item>
        <detail-list-item term="清关方式">{{ fbaDtail.customsClearance_dictText}}</detail-list-item>
        <detail-list-item term="交税方式">{{ fbaDtail.taxPayment_dictText}}</detail-list-item>
        <detail-list-item term="交货条款">{{ fbaDtail.address}}</detail-list-item>
        <detail-list-item term="VAT号">{{ fbaDtail.vat}}</detail-list-item>
        <detail-list-item term="参考号一">{{ fbaDtail.referenceNumber1}}</detail-list-item>
        <detail-list-item term="参考号二">{{ fbaDtail.referenceNumber2}}</detail-list-item>
        <detail-list-item term="备注">{{ fbaDtail.note}}</detail-list-item>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>


      <div class="title">货箱详情</div>
      <s-table
        style="margin-bottom: 24px"
        rowKey="id"
        :columns="goodsColumns"
        :data="loadGoodsData">
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>

      </s-table>

      <div class="title">退货进度</div>
      <s-table
        style="margin-bottom: 24px"
        :columns="scheduleColumns"
        :data="loadScheduleData">

        <template
          slot="status"
          slot-scope="status">
          <a-badge :status="status" :text="status | statusFilter"/>
        </template>

      </s-table>
    </a-card>

    <template slot="action">
<!--      <a-button-group >-->
<!--        <a-button type="primary" style="margin-right: 4px;" @click="handleExportXls('导入fba表')">导出箱单发票</a-button>-->
<!--        <a-button type="primary" style="margin-right: 4px;">导出唛头</a-button>-->
<!--        <a-button type="primary" style="margin-right: 4px;">导出装车清单</a-button>-->
<!--        <a-button type="primary" style="margin-right: 4px;">导出装箱通知书</a-button>-->
<!--        <a-button type="primary" style="margin-right: 4px;">导出报关资料</a-button>-->
<!--      </a-button-group>-->
      <a-button-group size="middle" style="margin-right: 4px;">
        <a-button icon="printer">打印标签</a-button>
        <a-button icon="edit">修改</a-button>
        <a-dropdown >
          <a-menu slot="overlay" @click="handleMenuClick" icon="cloud-download">
            <a-menu-item key="1">
              <a-icon type="cloud-download"  @click="handleExportXls('导入fba表')"/>导出发票
            </a-menu-item>
            <a-menu-item key="2">
              <a-icon type="cloud-download"/>导出运单
            </a-menu-item>
            <a-menu-item key="3" >
              <a-icon type="cloud-download"/>申报信息
            </a-menu-item>
          </a-menu>
          <a-button><a-icon type="cloud-download"/> 下载  </a-button>
        </a-dropdown>
        <a-button type="danger" > <a-icon type="close" />取消运单</a-button>
      </a-button-group>
    </template>
  </page-layout>

</template>

<script>

  import PageLayout from '@/components/page/PageLayout'
  import STable from '@/components/table/'
  import DetailList from '@/components/tools/DetailList'
  import ABadge from "ant-design-vue/es/badge/Badge"
  import { JeecgListMixin } from '../../../../mixins/JeecgListMixin'
  import { handleDetailss } from '../../../../api/manage'
  const DetailListItem = DetailList.Item

  export default {
    mixins:[JeecgListMixin],
    components: {
      PageLayout,
      ABadge,
      DetailList,
      DetailListItem,
      STable
    },
    data () {

      return {
        fbaDtail: {},
        goodsColumns: [
          {
            title: '货箱编号',
            dataIndex: 'caseid',
            key: 'caseid',
          },
          {
            title: '货箱重量',
            dataIndex: 'weight',
            key: 'weight',

          },
          {
            title: '货箱长度',
            dataIndex: 'length',
            key: 'length',

          },
          {
            title: '货箱高度',
            dataIndex: 'height',
            key: 'height',
          },
          {
            title: '货箱宽度',
            dataIndex: 'width',
            key: 'width',
          },
          {
            title: '中文名称',
            dataIndex: 'cnName',
            key: 'cnName',
          },
          {
            title: '英文名称',
            dataIndex: 'enName',
            key: 'enName',
          },
          {
            title: '申报单价',
            dataIndex: 'declaredPrice',
            key: 'declaredPrice',
          },
          {
            title: '申报数量',
            dataIndex: 'declaredNumber',
            key: 'declaredNumber',
          },
          {
            title: '产品海关编码',
            dataIndex: 'hscode',
            key: 'hscode',
          },
          {
            title: '产品材料',
            dataIndex: 'material',
            key: 'material',
          },
          {
            title: '产品用途',
            dataIndex: 'application',
            key: 'application',
          },
          {
            title: '品牌',
            dataIndex: 'brand',
            key: 'brand',
          },
          {
            title: '品牌类型',
            dataIndex: 'type',
            key: 'type',
          },
          {
            title: '产品型号',
            dataIndex: 'model',
            key: 'model',
          },
          {
            title: '销售链接',
            dataIndex: 'link',
            key: 'link',
          },
          {
            title: '产品售价',
            dataIndex: 'price',
            key: 'price',
          },
          {
            title: '图片链接',
            dataIndex: 'picture',
            key: 'picture',
            scopedSlots: {customRender: 'imgSlot'}
          },
        ],
        // 加载数据方法 必须为 Promise 对象
        loadGoodsData: () => {
          return new Promise((resolve => {
            resolve({
              data: this.dataList,
              pageSize: 10,
              pageNo: 1,
              totalPage: 1,
              totalCount: 10
            })
          })).then(res => {
            // console.log(res)
            return res
          })
        },
        loadData: () => {
          return this.$http.get('/zmexpress/zmImportFba/queryZmImportGoodByMainId', {
            params: this.$route.query.record.id
          }).then(res => {
            console.log(res.result)
            return res.result
          })
        },

        scheduleColumns: [
          {
            title: '时间',
            dataIndex: 'time',
            key: 'time'
          },
          {
            title: '当前进度',
            dataIndex: 'rate',
            key: 'rate'
          },
          {
            title: '状态',
            dataIndex: 'status',
            key: 'status',
            scopedSlots: { customRender: 'status' },
          },
          {
            title: '操作员ID',
            dataIndex: 'operator',
            key: 'operator'
          },
          {
            title: '耗时',
            dataIndex: 'cost',
            key: 'cost'
          }
        ],
        loadScheduleData: () => {
          return new Promise((resolve => {
            resolve({
              data: [
                {
                  key: '1',
                  time: '2017-10-01 14:10',
                  rate: '联系客户',
                  status: 'processing',
                  operator: '取货员 ID1234',
                  cost: '5mins'
                },
                {
                  key: '2',
                  time: '2017-10-01 14:05',
                  rate: '取货员出发',
                  status: 'success',
                  operator: '取货员 ID1234',
                  cost: '1h'
                },
                {
                  key: '3',
                  time: '2017-10-01 13:05',
                  rate: '取货员接单',
                  status: 'success',
                  operator: '取货员 ID1234',
                  cost: '5mins'
                },
                {
                  key: '4',
                  time: '2017-10-01 13:00',
                  rate: '申请审批通过',
                  status: 'success',
                  operator: '系统',
                  cost: '1h'
                },
                {
                  key: '5',
                  time: '2017-10-01 12:00',
                  rate: '发起退货申请',
                  status: 'success',
                  operator: '用户',
                  cost: '5mins'
                }
              ],
              pageSize: 10,
              pageNo: 1,
              totalPage: 1,
              totalCount: 10
            })
          })).then(res => {
            return res
          })
        },

        superFieldList:[],
        dataList: []
      }
    },
    created() {
      this.getList();
      this.getCaseDetails();
    },
    mounted() {

    },

    methods:{
      getList(){
         let fbaDtail={};
        fbaDtail = this.$route.query.record
        this.fbaDtail = fbaDtail;
        var params = fbaDtail.id
      },
      async getCaseDetails(){
        var list = '/zmexpress/zmImportFba/queryZmImportGoodByMainId'
        let params = this.$route.query.record.id
        this.dataList =await handleDetailss(list, {id: params}).then(res => {
          if (res.success) {
            return res.result;
          }
        })

      },
      show(){
         console.log(this.dataList)
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          'processing': '进行中',
          'success': '完成',
          'failed': '失败'
        }
        return statusMap[status]
      }
    },
    computed: {
      title () {
        var title = this.$route.query.record.fbaid + '/' +this.$route.query.record.orderid + '/' + this.$route.query.record.nameSender;
        return title;
      }
    },

  }
</script>

<style lang="less" scoped>
  .title {
    color: rgba(0,0,0,.85);
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 16px;
  }
</style>