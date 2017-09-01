<template>
  <div id="app">
    <view-box>
      <x-header slot="header" class="my-header">
        <div slot="overwrite-left">直播</div>
        <div>网易</div>
        <div slot="right">搜索</div>
      </x-header>

      <tab active-color="#f33" class="my-tab">
        <tab-item selected >新闻</tab-item>
        <tab-item>体育</tab-item>
        <tab-item>科技</tab-item>
        <tab-item>娱乐</tab-item>
        <tab-item>IT</tab-item>
        <tab-item>生活</tab-item>
      </tab>
      
      <scroller 
      class="my-scroller" 
      :on-refresh="refresh" 
      :on-infinite="infinite"
      ref="myScroller"
      >
        <swiper :list="slideList" loop auto :interval=3000></swiper>

        <marquee class="my-marquee">
          <marquee-item v-for="(item,index) in liveList" :key="index">{{item.title}}</marquee-item>
        </marquee>

        <panel :list="dataList" class="my-panel"></panel>
        <panel :list="moreList" class="my-panel"></panel>

      </scroller>

      <tabbar slot="bottom">
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_button.png" />
          <span slot="label">Wechat</span>
        </tabbar-item>
        <tabbar-item show-dot>
          <img slot="icon" src="./assets/icon_nav_msg.png" />
          <span slot="label">Message</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_article.png" />
          <span slot="label">Explore</span>
        </tabbar-item>
      </tabbar>
    </view-box>
    <router-view></router-view>
  </div>
</template>

<script>
import {ViewBox, XHeader, Tabbar, TabbarItem, Tab, TabItem, Swiper, Marquee, MarqueeItem, Panel} from 'vux'

let refreshKey = ['A', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B010']
let refreshIndex = 0
let refreshData = refreshKey[refreshIndex]
let start = 0
let page = 10
let end = start + 10
let upRefreshOnOff = false


function getRefreshKey(){
  refreshIndex ++
  if( refreshIndex >= refreshKey.length ){
    refreshIndex = 0
  }
  refreshData = refreshKey[refreshIndex]
}

export default {
  name: 'app',
  components:{
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Tab,
    TabItem,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  data(){
    // let dataList = []
    // for( let i=0; i<10; i++ ){
    //   dataList.push({
    //     src: 'http://placeholder.qiniudn.com/60x60/3cc51f/ffffff',
    //     title: '标题一',
    //     desc: '由各种物质组成的巨型球状天体，叫做星球。星球有一定的形状，有自己的运行轨道。',
    //     url: '/component/cell'
    //   })
    // }
    return{
      slideList:[],
      dataList:[],
      liveList:[],
      moreList:[]
    }
  },
  methods:{
    refresh(){
      //下拉刷新
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html',{
          miss: '00',
          refresh: refreshData
      }).then(data=>{
        if( data.code === 200 ){
          this.dataList = data.list.filter(item=>{
            return item.addata === null
          }).map(item=>{
            return {
              title:item.title,
              src:item.picInfo[0].url,
              desc:item.digest
            }
          })
          getRefreshKey()
          this.$refs.myScroller.finishPullToRefresh()
          this.$vux.toast.show({
            text:`刷新了${this.dataList.length}条信息`
          })
        }
      })
      console.log('下拉刷新')
    },
    //上啦刷新
    infinite(){
      if( !upRefreshOnOff ){
        this.$refs.myScroller.finishInfinite();
        return
      }
      console.log('上拉刷新')
      start += page
      end += page
      this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`,{
          miss: '00',
          refresh: refreshData
      }).then(data=>{
        if( data.code === 200 ){
          this.moreList = data.list.filter(item=>{
            return item.addata === null && item.picInfo.length
          }).map(item=>{
            return {
              title:item.title,
              src:item.picInfo[0].url,
              desc:item.digest
            }
          }).slice(1)

          this.$refs.myScroller.finishInfinite();
          this.$vux.toast.show({
            text:`新增了${end-start}条信息`
          })
          console.log(this.moreList)
        }
      })
    }
  },
  created(){
    this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html').then(data=>{
      console.log(data)

      //获取slideList
      this.slideList = data.focus.filter(item=>{
        return item.addata === null
      }).map(item=>{
        return {
          title:item.title,
          img:item.picInfo[0].url,
          url:item.link
        }
      }).slice(0,4)

      //获取dataList
      this.dataList = data.list.filter(item=>{
        return item.addata === null
      }).map(item=>{
        return {
          title:item.title,
          src:item.picInfo[0].url,
          desc:item.digest
        }
      })

      //获取liveList
      this.liveList = data.live.filter(item=>{
        return item.addata === null
      }).map(item=>{
        return {
          title: item.title
        }
      })

      upRefreshOnOff = true
    })
  }
}
</script>

<style lang="less">
  @import '~vux/src/styles/reset.less';
  *{
    margin: 0;
    padding: 0;
  }
  html,body{
    height:100%;
    width:100%;
    overflow: hidden;
  }
  #app{
    height:100%;
    .my-header{
      width: 100%;
      background-color: #f33;
      position: absolute;
      left: 0;
      top: 0;
      z-index: 10;
    }
    .my-tab{
      position: absolute;
      top: 46px; 
      left: 0;
      width: 100%;
      z-index: 10;
    }
    .weui-panel{
      margin-top: 0;
      .weui-media-box__hd,.weui-media-box__hd img{
        width: 82px;
        height: 60px;
      }
    }
    .my-scroller{
      position: relative;
      top: 80px;
      padding-bottom: 70px;
    }
    .my-marquee{
      padding: 0 10px;
    }
  }
</style>
