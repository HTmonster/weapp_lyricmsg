<template>
  <div >

    <div class="logoinfo">
      <img class="logo" src="/static/images/logo.png"  />
    </div>

    <van-search
      placeholder="请输入搜索的词语"
      @search="onSearch"
    >
    </van-search>
    <van-toast id="van-toast" />
    <van-notify id="van-notify" />

    <div>
      <van-cell-group>
        <van-transition name="slide-up"  >
          <span v-for="item in querys" :key="item.songid">
            <van-cell  v-bind:title="item.lyric"
                       v-bind:value="'《'+item.title+'》'+item.author"
                       is-link
                       v-bind:url="'/pages/share/main?id='+item.songid+'&w='+search_w"/>
          </span>

        </van-transition>
      </van-cell-group>
    </div>

  </div>
</template>

<script>
import Toast from 'vant-weapp/dist/toast/toast';
import Notify from 'vant-weapp/dist/notify/notify';


export default {
  data () {
    return {
      search_w:"",
      querys:[]
    }
  },


  methods: {
    onSearch(ev){
      console.log('search:', ev.mp.detail)
      this.search_w=ev.mp.detail
      Toast.loading({
        mask: false,
        message: '加载中...',
        loadingType: "spinner"
      });

      this.$http.get("/api/lyric/"+this.search_w).then((d)=>{
        //日志
        console.log("请求成功",d.data)
        if(d.data.status==="OK"){
          //请求成功
          //console.log(this.querys)
          this.querys=d.data.data
          //console.log(this.querys)
          Toast.clear()
          if(this.querys.length===0){
            Notify({text:'抱歉没有搜索到，正在完善数据库中！',backgroundColor:"#f44"});
          }
          //缓存
          this.cache()
        }
      }).catch(err=>{
        console.log(err.status,err.message)
        Toast.clear()
        Toast.fail({
          message: '抱歉，搜索出现问题',
        });
      })
    },
    cache(){
      try{
        wx.setStorage({
          key:'querys',
          data:this.querys,
          success:()=>{
            console.log("缓存成功")
          }
        })
      }
      catch(e){}
    }
  },


  // mounted:function () {
  //   try{
  //     wx.setStorage({
  //       key:'querys',
  //       data:this.querys,
  //       success:()=>{
  //         console.log("缓存成功")
  //       }
  //     })
  //   }
  //   catch(e){}
  // }


}
</script>

<style scoped>
.logoinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.logo {
  width: 200px;
  height: 200px;
}

</style>
