<template>
  <div class="order_list">
    <nav-bar title="订单列表">
      <span style="color:#333">
        <svg-icon icon-class="share" :width="15" :height="15" />
      </span>
    </nav-bar>
    <van-tabs
      v-model="activeIndex"
      :swipe-threshold="5"
      sticky
      swipeable
      offset-top="46"
      type="line"
      animated
      @click="handleTabClick"
    >
      <van-tab v-for="(tabTitle, index) in tabTitles" :key="index" :title="tabTitle">
        <van-pull-refresh
          v-model="refreshing"
          style="min-height: calc(100vh - 84px)"
          @refresh="onRefresh"
        >
          <van-list
            v-model="loading"
            :finished="finished"
            :immediate-check="false"
            finished-text="没有更多了"
            @load="getOrderList"
          >
            <van-panel
              v-for="(el, i) in orderList"
              :key="i"
              :title="'订单编号: ' + el.orderSn"
              :status="el.orderStatusText"
            >
              <van-card
                v-for="(goods, goodsI) in el.goodsList"
                :key="goodsI"
                :title="goods.goodsName"
                :num="goods.number"
                :thumb="goods.picUrl"
                @click.native="toOrderDetail(goods.id)"
              >
                <div slot="desc">
                  <div class="desc">
                    <van-tag
                      v-for="(spec, idx) in goods.specifications"
                      :key="idx"
                      plain
                      style="margin-right:6px;"
                    >{{ spec }}</van-tag>
                  </div>
                </div>
              </van-card>
              <div class="total">合计: {{ el.actualPrice | yuan }}（含运费{{ el.post_fee | yuan }}）</div>

              <div slot="footer" class="footer_btn">
                <van-button
                  v-if="el.handleOption.cancel"
                  size="small"
                  @click.stop="cancelOrder(el.id)"
                >取消订单</van-button>
                <van-button
                  v-if="el.handleOption.pay"
                  size="small"
                  type="danger"
                  @click.stop="toPay(el.id)"
                >去支付</van-button>
                <van-button
                  v-if="el.handleOption.refund"
                  size="small"
                  type="danger"
                  @click.stop="refundOrder(el.id)"
                >退款</van-button>
                <van-button
                  v-if="el.handleOption.confirm"
                  size="small"
                  type="danger"
                  @click.stop="confirmOrder(el.id)"
                >确认收货</van-button>
                <van-button
                  v-if="el.handleOption.delete"
                  size="small"
                  @click.stop="delOrder(el.id)"
                >删除订单</van-button>
                <van-button
                  v-if="el.handleOption.comment"
                  size="small"
                  @click.stop="commentOrder(el.id)"
                >去评价</van-button>
              </div>
            </van-panel>
          </van-list>
        </van-pull-refresh>
      </van-tab>
    </van-tabs>
  </div>
</template>

<script>
import {
  orderList,
  orderDelete,
  orderConfirm,
  orderCancel,
  orderRefund
} from '@/api/order'

export default {
  name: 'OrderList',

  props: {
    active: {
      type: [String, Number],
      default: 0
    }
  },
  data() {
    return {
      activeIndex: Number(this.active),
      tabTitles: ['全部', '待付款', '待发货', '待收货', '已完成'],
      orderList: [],
      refreshing: false,
      page: 0,
      limit: 10,
      loading: false,
      finished: false
    }
  },
  created() {
    this.init()
  },

  methods: {
    init() {
      this.page = 0
      this.orderList = []
      this.getOrderList()
    },
    // 下拉刷新
    onRefresh() {
      this.refreshing = true
      this.pageNum = 1
      this.getOrderList()
    },
    getOrderList() {
      this.page++
      orderList({
        showType: this.activeIndex,
        pageNum: this.page,
        pageSize: this.limit,
        sortName: 'updateTime,createTime'
      }).then((res) => {
        this.orderList.push(...res.map.data)
        this.loading = false
        this.refreshing = false
        this.finished = res.map.page >= res.map.pages
      })
    },
    delOrder(id) {
      this.$dialog
        .confirm({ message: '确定要删除该订单吗?' })
        .then(() => {
          orderDelete(id).then(() => {
            this.init()
            this.$toast('已删除订单')
          })
        })
        .catch(() => {})
    },
    cancelOrder(id) {
      this.$dialog
        .confirm({ message: '确定要取消该订单吗?' })
        .then(() => {
          orderCancel(id).then(() => {
            this.init()
            this.$toast('已取消该订单')
          })
        })
        .catch(() => {})
    },
    refundOrder(id) {
      this.$dialog
        .confirm({ message: '确定要申请退款吗?' })
        .then(() => {
          orderRefund(id).then(() => {
            this.init()
            this.$toast('已申请订单退款')
          })
        })
        .catch(() => {})
    },
    confirmOrder(id) {
      this.$dialog
        .confirm({
          message: '请确认收到货物, 确认收货后无法撤销!'
        })
        .then(() => {
          orderConfirm(id).then(() => {
            this.init()
            this.$toast('已确认收货')
          })
        })
        .catch(() => {})
    },
    commentOrder(id) {},
    toPay(id) {
      this.$router.push({ name: 'OrderPay', params: { orderId: id }})
    },
    handleTabClick() {
      this.page = 0
      this.orderList = []
      this.getOrderList()
    },
    toOrderDetail(id) {
      this.$router.push({
        path: `/detail/${id}`
      })
    }
  }
}
</script>
<style lang="scss">
.order-list {
  .van-card {
    background: #fff;
    margin-top: 0;
  }
  .van-tabs__wrap {
    background: #fff;
    padding: 4px 0;
  }
}
</style>
<style lang="scss" scoped>
.order_list {
  background: #f5f5f5;
  .list-item {
    width: 700px;
    margin: 0 auto;
    margin-top: 12px;
    border-radius: 10px;
    overflow: hidden;
    background: #fff;
  }
  .van-panel {
    margin-top: 20px;
  }

  .van-card {
    background-color: #fff;
  }

  .total {
    text-align: right;
    padding: 10px;
  }

  .footer_btn {
    text-align: right;
    .van-button {
      margin-left: 10px;
    }
  }
}
</style>
