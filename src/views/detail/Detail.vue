<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">
      <detail-swiper :to-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <detail-bottom-bar/>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
import Scroll from 'components/common/scroll/Scroll'

import GoodsList from 'components/content/goods/GoodsList'
import BackTop from 'components/content/backTop/BackTop'

import DetailNavBar from './childComps/DetailNavBar'
import DetailSwiper from './childComps/DetailSwiper'
import DetailBaseInfo from './childComps/DetailBaseInfo'
import DetailShopInfo from './childComps/DetailShopInfo'
import DetailGoodsInfo from './childComps/DetailGoodsInfo'
import DetailParamInfo from './childComps/DetailParamInfo'
import DetailCommentInfo from './childComps/DetailCommentInfo'
import DetailBottomBar from './childComps/DetailBottomBar'

import { getDetail, getRecommend, Goods, Shop, GoodsParam } from 'network/detail'
import {itemListenerMixin} from 'common/mixin'
import {debounce} from 'common/utils'

export default {
  name: 'Detail',
  components: {
    Scroll,
    GoodsList,
    BackTop,

    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    // 参数
    DetailParamInfo,
    DetailCommentInfo,
    DetailBottomBar
  },
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themeTopYs: [],
      getThemeTopY: null,
      currentIndex: 0,
      isShowBackTop: false,

    }
  },
  mixins: [itemListenerMixin],
  created() {
    // 1. 保存传入的iid
    this.iid = this.$route.params.iid

    // 2. 根据iid请求111数据
    getDetail(this.iid).then(res => {
      // console.log(res);
      const data = res.result
      // 1. 获取商品轮播图片
      this.topImages = res.result.itemInfo.topImages

      // 2. 获取商品信息
      this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)

      // 3. 创建店铺信息
      this.shop = new Shop(data.shopInfo)

      // 4. 保存商品的详细数据
      this.detailInfo = data.detailInfo

      // 5. 获取参数的信息
      this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)

      // 6. 获取评论信息
      if(data.rate.cRate != 0) {
        this.commentInfo = data.rate.list[0]
      }
    })
    // 3. 请求推荐数据
    getRecommend().then(res => {
      // console.log(res);
      this.recommends = res.data.list
    })

    // 4. 渲染完成
    this.getThemeTopY = debounce(() => {
      this.themeTopYs = []
      this.themeTopYs.push(0)
      this.themeTopYs.push(this.$refs.params.$el.offsetTop)
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
      // Number.MAX_VALUE  // 最大值
      this.themeTopYs.push(Number.MAX_VALUE)
      console.log('---');
    })
  },
  destroyed() {
    this.$bus.$off('itemImageLoad', this.itemImgListener)
  },
  methods: {
    imageLoad() {
      this.$refs.scroll.refresh()
      this.getThemeTopY()
    },
    // 点击标题滚动到相应位置
    titleClick(index) {
      // console.log(index);
      
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
    },
    contentScroll(position) {
      // 获取y值
      const positionY = -position.y
      let length = this.themeTopYs.length
      for (let i = 0; i < length - 1; i++) {
        if ((this.currentIndex !== i) && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1]) {
          this.currentIndex = i;
          this.$refs.nav.currentIndex = this.currentIndex
        }


        // 使用的话上面的遍历不需要-1
        // if ( (this.currentIndex !== i) && 
        //     (i < length - 1 && 
        //     positionY >= this.themeTopYs[i] && 
        //     positionY < this.themeTopYs[i+1]) || 
        //     (i === length - 1 && 
        //     positionY >= this.themeTopYs[i])) {
        //   this.currentIndex = i;
        //   this.$refs.nav.currentIndex = this.currentIndex
        // }
      }

      this.isShowBackTop = (-position.y) > 800
    },
    // 返回顶部
    backClick() {
      // console.log('返回顶部');
      this.$refs.scroll.scrollTo(0, 0)
    }

  }
}
</script>

<style scoped>
  #detail {
    position: relative;
    background-color: #fff;
    height: 100vh;
    z-index: 9;
  }
  .detail-nav {
    position: relative;
    background-color: #fff;
    z-index: 9;
  }
  .content {
    height: calc(100% - 44px - 49px);
    overflow: hidden;
  }
</style>