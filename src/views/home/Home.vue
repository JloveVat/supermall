<template>
  <div id="home">
    <!-- 顶部导航条 -->
    <nav-bar class="home-nav">
      <div slot="center">
        购物街
      </div>
    </nav-bar>
    <!-- 准备吸顶的导航 -->
    <tab-control 
          :titles='["流行", "新款", "精选"]'
          @tabClick="tabClick"
          ref="tabControl1"
          class="tab-control"
          v-show="isTabFixed"/>
    <!-- 滚动 -->
    <scroll class="content" 
            ref="scroll" 
            :probe-type="3" 
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <!-- 轮播图 -->
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <!-- 推荐 -->
      <recommend-view :recommends="recommends"></recommend-view>
      <!-- 本周流行 -->
      <feature-view/>
      <!-- 中间导航 -->
      <tab-control 
          :titles='["流行", "新款", "精选"]'
          @tabClick="tabClick"
          ref="tabControl2"/>
      <!-- 商品列表 -->
      <goods-list :goods="showGoods"/>
    </scroll>
      <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
import NavBar from 'components/common/navbar/NavBar'
import Scroll from 'components/common/scroll/Scroll'

import TabControl from 'components/content/tabControl/TabControl'
import GoodsList from 'components/content/goods/GoodsList'
import BackTop from 'components/content/backTop/BackTop'

import HomeSwiper from './childComps/HomeSwiper'
import RecommendView from './childComps/RecommendView'
import FeatureView from './childComps/FeatureView'

import { getHomeMultidata, getHomeGoods } from 'network/home'
import {itemListenerMixin} from 'common/mixin'

export default {
  components: {
    NavBar,
    Scroll,

    TabControl,
    GoodsList,
    BackTop,

    HomeSwiper,
    RecommendView,
    FeatureView
  },
  mixins: [itemListenerMixin],
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      },
      currentType: 'pop',
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0,
    }
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    }
  },
  activated() {
    // console.log('activated');
    this.$refs.scroll.scrollTo(0, this.saveY, 0)
    this.$refs.scroll.refresh()
  },
  deactivated() {
    // console.log('deactivated');
    this.saveY = this.$refs.scroll.getScrollY()

    // 2. 取消全局事件的监听
    this.$bus.$off('itemImageLoad', this.itemImgListener)
  },

  created() {
    // 1. 请求多个数据
    this.getHomeMultidata()

    // 2. 请求商品数据
    this.getHomeGoods('pop')
    this.getHomeGoods('new')
    this.getHomeGoods('sell')

    
  },
  methods: {
    // 网络请相关方法
    // 1. 请求多个数据
    getHomeMultidata() {
      getHomeMultidata().then(res => {
      // console.log(res);
      this.banners = res.data.banner.list
      this.recommends = res.data.recommend.list
      })
    },
    // 2. 请求商品数据
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1

        this.$refs.scroll.finishPullUp()
      })
    },

    // 轮播图加载完成
    swiperImageLoad() {
      // 获取tabControl的offsetTop
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      // console.log(this.tabOffsetTop);
    },
    // 事件监听方法
    tabClick(index) {
      // console.log(index);
      switch(index) {
        case 0:
          this.currentType = 'pop'
          break;
        case 1:
          this.currentType = 'new'
          break;
        case 2:
          this.currentType = 'sell'
          break;
      }
      this.$refs.tabControl1.currentIndex = index
      this.$refs.tabControl2.currentIndex = index
    },
    // 内容滚动
    contentScroll(position) {
      // 1. 判断BackTop是否显示
      // console.log(position);
      this.isShowBackTop = (-position.y) > 800
      // 2. 决定tabControl是否吸顶
      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    // 上拉加载更多
    loadMore() {
      // console.log('加载更多');
      this.getHomeGoods(this.currentType)
    },
    // 返回顶部
    backClick() {
      // console.log('返回顶部');
      this.$refs.scroll.scrollTo(0, 0)
    },

  }
}
</script>

<style scoped>
  #home {
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    /* position: fixed;
    top: 0;
    right: 0;
    left: 0;
    z-index: 9; */
  }
  /* 吸顶的导航条 */
  .tab-control {
    position: relative;
    z-index: 9;
  }

  .content {
    height: calc(100% - 93px);
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
  /* 第二种方式 */
  /* .content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
</style>