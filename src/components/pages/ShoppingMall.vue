<template>
  <div>
    <!--search bar layout-->
    <van-search
      v-model.trim="searchValue"
      placeholder="请输入搜索关键词"
      show-action
      background="#e5017d"
      @search="search"
    >
      <div slot="action" @click="search" class="search-txt">搜索</div>
    </van-search>
    <!--swipwer area-->
    <div class="swiper-area">
      <van-swipe :autoplay="2000">
        <van-swipe-item v-for="(banner,index) in bannerPicArray" :key="index" @click.native="goGoodsPage(banner.ID)">
          <img v-lazy="banner.IMAGE1" class="swipe-img"/>
        </van-swipe-item>
      </van-swipe>
    </div>
    <div class="type-bar">
      <div
        v-for="(cate,index) in category"
        :key="index"
        class="type-item"
        @click="$router.push(`/main/cateList?cateId=${cate.ID}`)"
      >
        <img v-lazy="cate.IMAGE" width="90%" />
        <span>{{cate.MALL_CATEGORY_NAME}}</span>
      </div>
    </div>
    <!--Recommend goods area-->
    <div class="recommend-area">
      <div class="recommend-title">
        商品推荐
      </div>
      <div class="recommend-body">
        <!--swiper-->
        <swiper :options="swiperOption">
          <swiper-slide v-for=" (item ,index) in recommendGoods" :key="index">
            <div class="recommend-item" @click="goGoodsPage(item.ID)">
              <img :src="item.IMAGE1" width="80%" />
              <div>{{item.NAME}}</div>
              <div>￥{{item.ORI_PRICE}} (￥{{item.PRESENT_PRICE}})</div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
    </div>
    <floor-component :floor1="floor1" :floor-title="floorName.floor1"></floor-component>
    <floor-component :floor1="floor2" :floor-title="floorName.floor2"></floor-component>
    <floor-component :floor1="floor3" :floor-title="floorName.floor3"></floor-component>
    <!--Hot Area-->
    <div class="hot-area">
      <div class="hot-title">热卖商品</div>
      <div class="hot-goods">
        <van-list>
          <van-row gutter="20">
            <van-col span="12" v-for="(item,index) in hotGoods" :key="index">
              <goods-info :item="item" @click.native="goGoodsPage(item.ID)"></goods-info>
            </van-col>
          </van-row>
        </van-list>
      </div>
    </div>
  </div>
</template>
<script>
import FloorComponent from '../FloorComponent'
import GoodsInfo from '../GoodsInfo'
import { toMoney } from '@/utils/moneyFilter.js'
import Api from '@/api/api'
export default {
  components: {
    FloorComponent,
    GoodsInfo
  },
  data () {
    return {
      searchValue: '',
      bannerPicArray: [], // 轮播图片
      recommendGoods: [], // 推荐商品列表
      swiperOption: {
        autoplay: true,
        spaceBetween: 9,
        slidesPerView: 3,
        speed: 1000,
        loop: true
      },
      hotGoods: [], // 热卖商品
      floor1: [
        {
          image: ''
        },
        {
          image: ''
        },
        {
          image: ''
        }
      ], // 楼层1数据
      floor2: [], // 楼层2的数据
      floor3: [], // 楼层3的数据
      floorName: '', // 楼层标题
      category: [] // 商品种类列表
    }
  },
  filters: {
    moneyFilter (money) {
      return toMoney(money)
    }
  },
  created () {
    this.floor2 = this.floor3 = this.floor1
    this.$toast.loading({
      mask: true,
      duration: 0,
      message: '数据加载中...'
    })
    Promise.all([this.getInitData(), this.getFloorData()]).finally(_ => {
      this.$toast.clear()
    })
  },
  methods: {
    search () {
      this.$router.push(`/goodsList/${this.searchValue}`)
    },
    // 去商品详情页
    goGoodsPage (goodsId) {
      this.$router.push(`/goods/${goodsId}`)
    },
    // 获取初始化信息
    async getInitData () {
      await Api.getInitData().then(data => {
        this.category = data.data.category
        this.bannerPicArray = data.data.slides
        this.recommendGoods = data.data.recommend
        this.hotGoods = data.data.hotGoods
      })
    },
    // 获取楼层数据
    async getFloorData () {
      await Api.getGoodsInfo().then(data => {
        this.floor1 = data.data.floor1
        this.floor2 = data.data.floor2
        this.floor3 = data.data.floor3
        this.floorName = data.data.floorName
      })
    }
  }
}
</script>
<style scoped lang="scss">
  .swipe-img{
    width: 100%;
    height: 167px;
  }
  .search-txt{
    color: #ffffff;
  }
  .hot-area{
    text-align: center;
    font-size:14px;
    height: 1.8rem;
    line-height:1.8rem;
  }
  .recommend-body{
    border-bottom: 1px solid #eee;
  }
  .recommend-item{
    border-right: 1px solid #eee;
    font-size: 12px;
    text-align: center;
  }
  .recommend-area{
    background-color: #fff;
    margin-top: .3rem;
  }
  .recommend-title{
    border-bottom:1px solid #eee;
    font-size:14px;
    padding:.2rem;
    color:#e5017d;
  }
  .type-item{
    width: 20%;
    overflow: hidden;
  }
  .van-row {
    margin: 0 !important;
  }
  .search-input{
    width:100%;
    height: 1.8rem;
    border-top:0;
    border-left:0;
    border-right:0;
    border-bottom: 1px solid #eeeeee !important;
    background-color: #e5017d;
    color:#fff;
  }
  .location-icon{
    padding-top: .2rem;
    padding-left: .3rem;
  }
  .search-btn{
    text-align: center;
  }
  .swiper-area{
    clear:both;
    overflow: hidden;
  }
  .type-bar{
    background-color: #fff;
    margin:0 .3rem .3rem .3rem;
    border-radius: .3rem;
    font-size:14px;
    display: flex;
    flex-direction:row;
    flex-wrap:nowrap;
  }
  .type-bar div{
    padding: .3rem;
    font-size: 12px;
    text-align: center;
  }
</style>
