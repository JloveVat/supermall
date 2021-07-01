<template>
  <div id="home">
    <!-- 顶部导航条 -->
    <nav-bar class="home-nav">
      <div slot="center">
        购物街
      </div>
    </nav-bar>
    <!-- 轮播图 -->
    <home-swiper :banners="banners"/>
    <!-- 推荐 -->
    <recommend-view :recommends="recommends"></recommend-view>
    <!-- 本周流行 -->
    <feature-view/>
    <!-- 导航 -->
    <tab-control class="tab-control" :titles='["流行", "新款", "精选"]'></tab-control>
    <!-- 商品列表 -->
    <goods-list :goods="goods['pop'].list"/>
    <ul>
      <li>内容</li>
      <li>内容</li>
      <li>内容</li>
      <li>1</li>
    </ul>
  </div>
</template>

<script>
import NavBar from 'components/common/navbar/NavBar'
import TabControl from 'components/content/tabControl/TabControl'
import GoodsList from 'components/content/goods/GoodsList'

import HomeSwiper from './childComps/HomeSwiper'
import RecommendView from './childComps/RecommendView'
import FeatureView from './childComps/FeatureView'

import { getHomeMultidata, getHomeGoods } from 'network/home'

export default {
  components: {
    NavBar,
    TabControl,
    GoodsList,

    HomeSwiper,
    RecommendView,
    FeatureView
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      }
    }
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
        this.goods[type].page++
      })
    }

  }
}
</script>

<style>
  #home {
    padding-top: 44px;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    position: fixed;
    top: 0;
    right: 0;
    left: 0;
    z-index: 9;
  }

  .tab-control {
    position:sticky;
    top: 44px;
  }
</style>