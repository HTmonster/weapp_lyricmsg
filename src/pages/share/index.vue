<template>
    <div style="text-align: center">

      <img  class="cover" v-bind:src="album_url" >

      <p v-if="isAllMatch" class="lyric">“{{value.lyric}}”</p>
      <p v-else class="lyric" style="color:gray;">"{{lyric_front}}<span style="color:black;">{{search_w}}</span>{{lyric_back}}"</p>
      <p class="info"> {{value.title}}-{{value.author}}</p>
      <p v-if="isAllMatch" class="location">{{value.time}}</p>
      <p v-else class="location">{{value.time}} <van-tag round  plain type="primary">{{index_s+1}}-{{index_e}}</van-tag></p>

      <div class="btn">
        <van-row>
          <van-col span="6" offset="1">
            <van-button icon="comment-circle-o" plain type="info" size="small" @click="onClickCopytext">复制文字</van-button>
          </van-col>

          <van-col span="6" offset="1">
            <van-button icon="photo-o" plain type="primary" size="small" @click="onClickSharePoster">分享海报</van-button>
          </van-col>

          <van-col span="9" offset="1">
            <van-button icon="friends-o"  type="danger" size="small" open-type="share">分享本页给好友</van-button>
          </van-col>
        </van-row>
      </div>

      <van-dialog
        use-slot
        title="编辑复制文字"
        v-bind:show="copy_dialog_show"
        show-cancel-button
        @cancel="onCancelCopy"
        @confirm="Copy2Clipboard"
      >
        <div style="width: 100%;">
          <textarea class="text" v-model="share_text"/>

        </div>
      </van-dialog>

      <van-toast id="van-toast" />

      <van-dialog
        use-slot
        title="分享海报"
        v-bind:show="share_poster_show"
        show-cancel-button
        style="text-align: center;"
        @cancel="onCancelPoster"
        @confirm="save2phone"
      >
        <canvas canvas-id="posterCanvas" v-bind:style="'margin:0px auto;width:'+windowWidth+'px;height:'+windowHeight+'px;'"></canvas>
      </van-dialog>

    </div>
</template>

<script>
import Toast from 'vant-weapp/dist/toast/toast';
import drawQrcode from '../../utils/weapp-qrcode.js'
export default {

  data () {
    return {
      //歌曲信息
      value:{},
      song_url:"",
      album_url:"",

      //搜索关键词
      search_w:"",

      //歌词匹配信息
      isAllMatch:true,
      lyric_front:'',
      lyric_back:'',
      index_s:-1,
      index_e:-1,

      //组件控制
      copy_dialog_show:0,
      share_poster_show:0,

      //分享信息
      share_text:"",
      poster:null,

      windowWidth: 0,
      windowHeight:0
    }
  },

  methods:{

    onClickCopytext(){
      this.copy_dialog_show=1;
    },
    Copy2Clipboard() {
      let self=this
      wx.setClipboardData({
        data: this.share_text,
        success(res){
          self.copy_dialog_show=0;
        },
        fail(res){
          Toast.fail('复制失败');
        },
      })
    },
    save2phone(){
      var that=this;

      wx.canvasToTempFilePath({
        canvasId: 'posterCanvas',
        success: function (res) {
          var canvasToTempFilePath = res.tempFilePath // 返回的图片地址保

          wx.saveImageToPhotosAlbum({
            filePath:canvasToTempFilePath,
            success(res) {
              that.share_poster_show=0;
            }
          })


        },
        fail: function () {
          Toast.fail('保存失败');
        }
      })
    },
    onCancelCopy(){
      this.copy_dialog_show=0;
    },
    onCancelPoster(){
      this.share_poster_show=0;
    },
    onClickSharePoster(){
      let that=this

      this.share_poster_show=1;
      //原底子图片信息
      wx.getImageInfo({
          src:"../../static/images/poster1.png",
          success: function(res) {
            var w,h;

            w=res.width
            h=res.height

            that.windowWidth=w
            that.windowHeight=h

            const ctx = wx.createCanvasContext("posterCanvas");

            ctx.drawImage("../../static/images/poster1.png",0,0,w,h)
            ctx.setFillStyle('#eee');
            ctx.setFontSize(15);

            // 绘制文字
            ctx.fillText("《"+that.value.title+"》· "+that.value.author, 80, 230)
            ctx.setFillStyle('#aaa');
            ctx.setFontSize(10);
            if(!that.isAllMatch){
              ctx.fillText(that.value.time+" ("+(that.index_s+1)+","+that.index_e+")", 90, 250)
            }else{
              ctx.fillText(that.value.time, 90, 250)
            }

            /*保存上下文 绘制 */


            drawQrcode({
              width:70,
              height:70,
              ctx:ctx,
              text:that.song_url,
              x:w-80,
              y:h-80
            })

            ctx.save();
            ctx.draw();

          },
          fail: function(res) {
            reject(res)
          }
        })
    }
  },

  mounted:function () {

    //设置分享
    wx.showShareMenu({
      withShareTicket: true
    })

    //获取传递的参数
    const pages = getCurrentPages()
    const currentPage = pages[pages.length - 1]
    const options = currentPage.options

    // 歌曲id
    this.songid=parseInt(options.id)
    this.search_w=options.w
    console.log(this.search_w)

    // 其他对应的值
    var querys=wx.getStorageSync('querys')
    for(var i=0, len = querys.length; i < len; i++) {
      if (this.songid === querys[i].songid){
        this.value=querys[i]
        break
      }
    }

    // 歌词位置搜索
    if(this.search_w!=this.value.lyric){
      this.isAllMatch=false

      var lyric=this.value.lyric
      var sub_index=lyric.indexOf(this.search_w)

      this.index_s=sub_index
      this.index_e=sub_index+this.search_w.length

      this.lyric_front=lyric.substr(0,this.index_s)
      this.lyric_back=lyric.substr(this.index_e,lyric.length)

      console.log(this.lyric_front)
      console.log(this.lyric_back)
    }

    //歌曲url
    this.album_url="http://y.gtimg.cn/music/photo_new/T002R180x180M000"+this.value.albummid.replace("\n","")+".jpg"

    console.log(this.album_url)
    this.song_url="https://i.y.qq.com/v8/playsong.html?songid="+this.value.songid

    //复制的文字
    this.share_text="我想说的话，藏在 "+this.value.author+"《"+this.value.title+"》 播放到"+this.value.time+"时候"
    if(!this.isAllMatch){
      this.share_text=this.share_text+" 第"+(this.index_s+1)+"到"+this.index_e+"个字"
    }
    this.share_text=this.share_text+"\n播放链接："+this.song_url

  },

  onShareAppMessage: function (ops) {
    return {
      title: this.value.author+"《"+this.value.title+"》"+this.value.time,
      path: `pages/share/index`,
      imageUrl:"http://y.gtimg.cn/music/photo_new/T002R180x180M000"+this.value.albummid+".jpg",
      success: function (res) {
        // 转发成功
        console.log("转发成功:" + JSON.stringify(res));
      },
      fail: function (res) {
        // 转发失败
        console.log("转发失败:" + JSON.stringify(res));
      }
    }
  },

}
</script>

<style>

.cover{
  width: 200px;
  height: 200px;
  padding-top: 20px;
}

.lyric{
    padding-top: 10px;
    font-size:30px;
}
.info{
  padding-top: 5px;
}
.location{
  padding-top: 5px;
  color:gray;
}
  .btn{
    padding-top: 40px;
  }
  .text{
    text-align: left;
    padding-top: 5px;
    margin: 2px auto;
    display: block;
    border: solid dodgerblue 2px;
    right: 2px;
    border-radius:2%;
  }
</style>
