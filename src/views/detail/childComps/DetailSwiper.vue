<template>
  <swiper class="detail-swiper">
    <swiper-item v-for="(item, index) in toImages" :key="index">
        <img :src="item" alt=""/>
    </swiper-item>
  </swiper>
</template>

<script>
import {Swiper, SwiperItem} from 'components/common/swiper'

export default {
  props: {
    toImages: {
      type: Array,
      default() {
        return []
      }
    }
  },
  components: {
    Swiper,
    SwiperItem
  },
}
</script>

<style>
  .detail-swiper {
    height: 250px;
    overflow: hidden;
  }
</style>