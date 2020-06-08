<template>
<div>
    <van-notify id="van-notify" />


    <van-row >
      <van-col span="5" offset="1">
        <p > LyricMsg</p>

      </van-col>

    </van-row >
    <van-row >
      <van-col span="23" offset="1">
        <p style="color:gray;font-size: 10px">   /*想说的话，藏在歌词里*/</p>
      </van-col>
    </van-row >
    <van-row id="info">
      <van-col span="5" offset="1">
        <van-tag plain type="primary">HTmonster</van-tag>
      </van-col>
      <van-col span="4" offset="1">
        <van-tag plain type="warning">傻吊出品</van-tag>
      </van-col>
      <van-col span="4" offset="1">
        <van-tag plain type="success">测试版本</van-tag>
      </van-col>
    </van-row>


  <div class="feedback">
    <textarea class="feedback-input" v-model="message">

    </textarea>
    <van-button type="primary"  size="small" @click="onSubmit">提交Bug或者改进建议</van-button>


  </div>
</div>

</template>

<script>
import Notify from 'vant-weapp/dist/notify/notify';

export default {

  data() {
    return {
      message:""
    }
  },
  methods: {

    onSubmit(){
      console.log(this.message)

      this.$http.post("/api/feedback/",{"content":this.message}).then((d)=>{
        //输出请求数据
        console.log(d.data)
        //输出响应头
        console.log(d.header)
      }).catch(err=>{
        console.log(err.status,err.message)
      })


      Notify({text:'提交成功，感谢您的支持',backgroundColor:"#07c160"});
    },

  }
}
</script>

<style>
  .feedback{
    width:100%;
    height: 250px;
    color:grey;
    margin-top: 40px;
    /*background-color: red;*/
    text-align: center;
  }
  .feedback .feedback-input{
    /*border: solid black 2px;*/
    border-radius: 3%;
    font-size: 15px;
    background-color: rgb(234,234,234);
    width: 90%;
    margin: 10px auto;
  }

</style>
