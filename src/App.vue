<template>
  <div id="app">
    <action-sheet></action-sheet>
<!--      一上来显示的这里是哪里?-->
    <transition :name="routerViewAnimation">
<!--        blurBgShow默认为false,当打开音乐详情会为true 隐藏-->
      <router-view v-show="!blurBgShow"></router-view>
    </transition>

    <search v-show="!blurBgShow" @searchshow="rankshow=false" @searchhide="rankshow=true"></search>
    <div class="content-warper" v-show="rankshow&&!blurBgShow">
<!--        https://github.com/surmon-china/vue-awesome-swiper
            vue中使用swiper-->
<!--        options:配置swiper的属性-->
      <swiper :options="swiperOption" class="swiper-box">
<!--          这是两个切换的item-->
        <swiper-slide class="swiper-item">
          <rank></rank>
        </swiper-slide>
        <swiper-slide class="swiper-item">
          <recommand></recommand>
        </swiper-slide>
<!--这里放了两个按钮
pagination:分页导航,默认小圆点-->
        <div class="swiper-pagination" slot="pagination"></div>
      </swiper>
    </div>
<!--      Vue 提供了 transition 的封装组件，在下列情形中
    条件渲染 (使用 v-if)
    条件展示 (使用 v-show)
    动态组件
    组件根节点
，可以给任何元素和组件添加进入/离开过渡
过渡后会更改routerViewAnimation,而这个是顶部transition的类名-->
<!--      transition的name属性是用来替换v-enter的v的,替换后匹配为play-slide-enter的css类名-->
    <transition name="play-slide" @after-enter="showBlurBg" @before-leave="hideBlurBg" @after-leave="routerViewAnimation='page-slide'">
<!--        歌曲详情页,点击底部播放会触发-->
      <play v-show="playPageShow"></play>
    </transition>
    <transition name="play-slide">
      <playing-list v-if="$store.state.NotifyService.playingList.show"></playing-list>
    </transition>

    <transition name="bar-slide">
      <div id="play-bar" v-show="!playPageShow">
        <!-- <audio id="music" :src="dataUrl" @timeupdate="updateTime" @ended="playContinue" autoplay></audio> -->
<!--          点击查看歌曲
打开上面的play-slide,触发钩子after-enter:showBlurBg-->
        <div class="play-bar-image-container" @touchstart="showPlayPage" @click="showPlayPage">
          <img class="play-bar-image" v-lazy="coverImgUrl">
        </div>
        <p class="play-bar-text" @touchstart="showPlayPage" @click="showPlayPage">{{song.name}}</p>
<!--          点击暂停或播放音乐-->
        <img class="play-bar-button" :src="playing?iconPause:iconPlay" @touchend="tapButton" @click="tapButton">
      </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
import Search from "./components/Search";
import Play from "./components/Play";
import Rank from "./components/Rank";
import Recommand from "./components/Recommand";
import ActionSheet from "./components/ActionSheet";
import PlayingList from "./components/PlayingList";

import { mapMutations, mapState, mapGetters } from "vuex";
import { swiper, swiperSlide } from "vue-awesome-swiper";

import 'swiper/dist/css/swiper.css'

const TAB_NAME = ["排行榜", "推荐"];

export default {
  components: {
    Search,
    Play,
    Rank,
    Recommand,
    ActionSheet,
    PlayingList,
    swiper,
    swiperSlide
  },
  methods: {
      // 播放按钮
    tapButton(event) {
        debugger;
      event.preventDefault();
      this.playing ? this.pause() : this.play();
    },
      // 当前播放音乐的封面
    showPlayPage(event) {
        debugger;
        event.preventDefault();
      this.playPageShow = true;
    },
    hidePlayPage(event) {
        debugger;
        event.preventDefault();
      this.playPageShow = false;
    },
    showBlurBg() {
        debugger;
        // name - string，用于自动生成 CSS 过渡类名。例如：name: 'fade' 将自动拓展为.fade-enter，.fade-enter-active等
      this.routerViewAnimation = "fade";
      this.blurBgShow = true;
    },
    hideBlurBg() {
        debugger;
      this.blurBgShow = false;
    },
    ...mapMutations(["play", "pause", "playContinue"])
  },
  data() {
    return {
      iconPlay: require("./assets/icon-play.png"),
      iconPause: require("./assets/icon-pause.png"),
      playPageShow: false,
      blurBgShow: false,
      rankshow: true,
      routerViewAnimation: "page-slide",
      swiperOption: {
        pagination: ".swiper-pagination",
        paginationClickable: true,
        paginationBulletRender(swiper, index, className) {
            // 返回两个按钮标签
          return `<span class="${className} swiper-pagination-bullet-custom">${TAB_NAME[
            index
          ]}</span>`;
          // return '<span class="' + className + ' swiper-pagination-bullet-custom' + '">' + (index + 1) + '</span>';
        }
      }
    };
  },
  computed: {
    ...mapGetters(["coverImgUrl"]),
    ...mapState({
      dataUrl(state) {
        return (
          "https://dl.stream.qqmusic.qq.com/C100" +
          state.PlayService.song.mid +
          ".m4a?fromtag=46"
        );
      },
      playing: state => state.PlayService.playing,
      song: state => state.PlayService.song
    })
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

html {
  overflow-x: hidden;
  background: #eeeeee;
}

body {
  display: flex;
  overflow-x: hidden;
}

#app {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC",
    "Helvetica Neue", STHeiti, "Microsoft Yahei", Tahoma, Simsun, sans-serif;
  width: 100%;
  height: 100%;
}

#app a {
  color: #42b983;
  text-decoration: none;
}

#play-bar {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 50px;
  background: -webkit-linear-gradient(top, #f9f9f9, #f3f3f3);
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  z-index: 2;
}

.play-bar-image-container {
  width: 37.5px;
  height: 37.5px;
  padding-left: 15px;
  cursor: pointer;
}

.play-bar-image {
  width: 37.5px;
  height: 37.5px;
  border-radius: 5px;
  display: inline-block;
}

.play-bar-text {
  flex-grow: 1;
  padding-left: 10px;
  cursor: pointer;
}

.play-bar-button {
  width: 20px;
  height: 20px;
  padding-right: 15px;
  cursor: pointer;
}

.page-slide-enter-active {
  /*transition: all 0.3s ease;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.page-slide-leave-active {
  /*transition: all 0.3s ease-out;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.page-slide-enter,
.page-slide-leave-active {
  transform: translateX(100%);
}

.fade-enter-active {
  /*transition: all 0.3s ease;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.fade-leave-active {
  /*transition: all 0.3s ease-out;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.fade-enter,
.fade-leave-active {
  opacity: 0;
}

.play-slide-enter-active {
  /*transition: all 0.3s ease;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.play-slide-leave-active {
  /*transition: all 0.3s ease-out;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.play-slide-enter,
.play-slide-leave-active {
  transform: translateY(100vh);
}

.bar-slide-enter-active {
  /*transition: all 0.3s ease;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.bar-slide-leave-active {
  /*transition: all 0.3s ease-out;*/
  transition: all 1s cubic-bezier(1 ,0 ,0.75 ,1);
}

.bar-slide-enter,
.bar-slide-leave-active {
  margin-bottom: -50px;
}

@media screen and (min-width: 68vh) {
  body {
    width: 68vh;
    margin: 0 auto;
  }

  #play-bar {
    width: 68vh;
  }
}

/*border-1px 部分*/

.border-1px {
  position: relative;
}

.border-1px-after:after {
  border-top: 1px solid #d0d0d0;
  content: " ";
  display: block;
  width: 100%;
  position: absolute;
  left: 0;
}

.border-1px-before:before {
  border-top: 1px solid #d0d0d0;
  content: " ";
  display: block;
  width: 100%;
  position: absolute;
  left: 0;
}

.border-1px:before {
  top: 0;
}

.border-1px:after {
  bottom: 0;
}

@media (-webkit-min-device-pixel-ratio: 1.5), (min-device-pixel-ratio: 1.5) {
  .border-1px:after,
  .border-1px:before {
    -webkit-transform: scaleY(0.7);
    -webkit-transform-origin: 0 0;
    transform: scaleY(0.7);
  }

  .border-1px:after {
    -webkit-transform-origin: left bottom;
  }
}

@media (-webkit-min-device-pixel-ratio: 2), (min-device-pixel-ratio: 2) {
  .border-1px:after,
  .border-1px:before {
    -webkit-transform: scaleY(0.5);
    transform: scaleY(0.5);
  }
}

@-webkit-keyframes imgFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes imgFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

img[lazy="loaded"] {
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
  -webkit-animation-name: imgFadeIn;
  animation-name: imgFadeIn;
}

img[lazy="error"] {
  border-radius: 2px;
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
  -webkit-animation-name: imgFadeIn;
  animation-name: imgFadeIn;
}

.content-warper {
  margin-top: 60px;
}

.swiper-box {
  width: 100%;
  height: 100%;
  margin: 0 auto;
}

.swiper-item {
  height: 100%;
}

.swiper-pagination-bullet-custom {
  width: 100% !important;
  height: 100% !important;
  margin: 0 !important;
  text-align: center;
  line-height: 50px;
  color: #999999;
  border-radius: 0 !important;
  background: #fff !important;
  opacity: 1 !important;
}

.swiper-pagination-bullet-custom.swiper-pagination-bullet-active {
  color: #000;
}

.swiper-pagination {
  top: 0;
  height: 50px;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  width: 100%;
}
</style>
