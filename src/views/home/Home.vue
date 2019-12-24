<template>
  <div id="home">
    <NavBar class="home-nav">
      <div slot="center">购物街</div>
    </NavBar>
    <scroll class="content" ref="scroll" :probe-type = "3" @scroll="contentScroll" :pull-up-load = "true" @pullingUp = "loadMore">
      <HomeSwiper :banners = "banners"></HomeSwiper>
      <recommend-view></recommend-view>
      <feature-view/>
      <tab-control class="tab-control" :titles = "['流行','新款','精选']" @tabClick = "tabClick"></tab-control>
      <goods-list :goods = "showGoods"></goods-list>
    </scroll>
    <BackTop @click.native="backClick" v-show = "isShowBack"></BackTop>
  </div>
</template>
<script>

import HomeSwiper from "./childrenComps/HomeSwiper"
import RecommendView from "./childrenComps/RecommendView"
import FeatureView from "./childrenComps/FeatureView"

import NavBar from "components/common/navbar/NavBar"
import TabControl from "components/content/tabControl/TabControl"
import GoodsList from "components/content/goods/GoodsList"
import BackTop from "components/content/backTop/BackTop"
import Scroll from "components/common/scroll/Scroll"

import {getHomeMultidata,getHomeGoods} from "network/home";
import { log } from 'util'


export default {
  name:'Home',
  components:{
    HomeSwiper,
    RecommendView,
    FeatureView,
    NavBar,
    TabControl,
    GoodsList,
    BackTop,
    Scroll
  },
  data(){
    return{
      banners:[],
      recommends:[],
      goods:{
        'pop':{page:0,list:[]},
        'new':{page:0,list:[]},
        'sell':{page:0,list:[]}
      },
      currentType:'pop',
      isShowBack:false,
    }
  },
  created(){
    this.getHomeMultidata();
    this.getHomeGoods('pop');
    this.getHomeGoods('new');
    this.getHomeGoods('sell');
  },
  computed:{
    showGoods(){
      return this.goods[this.currentType].list
    }
  },
  methods:{
    //事件监听相关方法
    tabClick(index){
      switch(index){
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      
    },
    backClick(){
      this.$refs.scroll.scrollTo(0,0,500);
    },
    contentScroll(position){
      this.isShowBack = -position.y>1000
      
    },
    loadMore(){
      this.getHomeGoods(this.currentType)
      
    },
    //网络请求相关方法
    getHomeMultidata(){
      getHomeMultidata().then(res=>{
      this.banners = res.data.banner.list;
      this.recommends = res.data.recommend.list;
    })
    },
    getHomeGoods(type){
      const page = this.goods[type].page + 1;
      getHomeGoods(type,page).then(res => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
        this.$refs.scroll.finishPullUp()
      })
    }
  }
}
</script>
<style scoped>
  #home {
    position: relative;
    padding-top: 44px;
    height: 100vh;
  }
  .home-nav {
    background-color: var(--color-tint);
    color:white;
    position: fixed;
    top:0;
    left:0;
    right: 0;
    z-index: 9999
  }
  .tab-control {
    position: sticky;
    top: 44px;
    z-index: 999;
  }
  .content {
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
</style>