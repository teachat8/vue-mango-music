<template>
  <div class="music-singer">
    <detail :title="title" :data="data" :loading="loading" />
    <toast :show="toastShow" content="加载失败..." />
  </div>
</template>

<script>
  import Toast from "@/common/Toast"
  import Detail from "../detail/Detail"
  import {getSingerInfo} from "@/api/singer"
  import {getSongVKey} from "@/api/song"
  import {CODE_SUCCESS} from "@/api/config"
  import * as SingerModel from "@/model/singer"
  import * as SongModel from "@/model/song"

  export default {
    name: "singer",
    data() {
      return {
        loading: true,
        title: "歌曲",
        data: {
          name: "",
          img: require("../../assets/imgs/play_bg.jpg"),
          desc: "",
          songs: []
        },
        toastShow: false
      };
    },
    methods: {
      getSongUrl(song, mId) {
        getSongVKey(mId).then((res) => {
          if (res) {
            if (res.code === CODE_SUCCESS) {
              if (res.data.items) {
                let item = res.data.items[0];
                song.url = `http://dl.stream.qqmusic.qq.com/${item.filename}?vkey=${item.vkey}&guid=3655047200&fromtag=66`
              }
            }
          }
        });
      }
    },
    created() {
      getSingerInfo(this.$route.params.singerId).then((res) => {
        // console.log("获取歌手详情：");
        if (res) {
          // console.log(res);
          if (res.code === CODE_SUCCESS) {
            let singer = SingerModel.createSingerByDetail(res.data);

            let songList = res.data.list;
            let songs = [];
            songList.forEach(item => {
              if (item.musicData.pay.payplay === 1) { return }
              let song = SongModel.createSong(item.musicData);
              //  获取歌曲vkey
              this.getSongUrl(song, song.mId);
              songs.push(song);
            });
            this.data = {
              name: singer.name,
              img: singer.img,
              desc: '',
              songs
            };
            this.loading = false;
          }
        }
      }).catch(() => {
        // 调用接口失败显示Toast，1秒后隐藏Loading组件
        this.toastShow = true;
        setTimeout(() => {
          this.toastShow = false;
          this.loading = false;
        }, 1000);
      });
    },
    components: {
      Toast,
      Detail
    }
  }
</script>

<style lang="stylus" scoped>
  .music-singer
    position: fixed
    top: 0
    left: 0
    right: 0
    bottom: 0
    z-index: 100
    /*background-color: #F8F8FF*/
</style>
