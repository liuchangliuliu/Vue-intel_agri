<template>
  <div class="con">
    <!--视频播放-->
    <div class="video">
      <!--视频播放的主体-->
      <div class="left">
        <video-player  class="video-player-box"
                       ref="videoPlayer"
                       :options="playerOptions"
                       :playsinline="true"
                       customEventName="customstatechangedeventname"

                       @play="onPlayerPlay($event)"
                       @pause="onPlayerPause($event)"
                       @ended="onPlayerEnded($event)"
                       @waiting="onPlayerWaiting($event)"
                       @statechanged="playerStateChanged($event)"
                       @ready="playerReadied">
        </video-player>
        <div class="title" v-model="videoInfo">
          <span class="name" >{{videoInfo.video_name}} :</span>
          <span class="desc">{{videoInfo.desc}}</span><br/>
          <span class="click_num"><img src="../../static/images/video_click_num.jpg">  {{videoInfo.click_num}}</span>
          <span class=type></span>
        </div>

      </div>
      <!--推荐视频列表-->
      <div class="right">
        <div class="recommend" v-for="(videoRecommendItem, index) in videoRecommend" v-if="index <6" @click="RecommendVideo(videoRecommendItem.id)">
          <div class="rec_img"><img :src='"http://118.24.116.137:8001"+videoRecommendItem.video_img'></div>
          <div class="content">
            <div class="content_title">{{videoRecommendItem.video_name}}:{{videoRecommendItem.desc}}</div>
            <div class="con_click_num">
              <img class="rec_click_img" src="../../static/images/video_click_num.jpg">
              <span class="rec_click_num">{{videoRecommendItem.click_num}}</span>
            </div>
          </div>
        </div>

      </div>

    </div>

    <!--评论-->
    <div class="commend">
      <!--创建自己的根评论-->
      <div >
       <input type="text" class="create_root_commend" placeholder="请写下自己的评论..." v-model="createRootCommend" />
        <input type="button" class="create_root_button" value="评论" @click="CreateRootCommend()"/>
      </div>


      <!--根评论-->
      <!--0-->
      <div class="root_commend" v-for="(root_com,i) in a" :key="i" >
        <div class="commend_up">
          <img class="root_commend_icon" :src="root_com.user.user_profile.image">
          <div class="root_commend_name">{{root_com.user.user_profile.nick_name}}
            <div class="root_commend_time">{{root_com.add_time}}</div>
          </div>
        </div>
        <div class="commend_content">{{root_com.comment}}
        </div>
        <div class="commend_down">
          <input type="text" class="reply_input" v-if="show_input[0][i].status" v-model="show_input[0][i].value">
          <input type="button" class="reply_button" v-if="show_input[0][i].status" value="评论" @click="Reply(root_com.id,root_com.id,root_com.user.id,1,0,i)"/>
          <input type="button" class="reply_cancle"  v-if="show_input[0][i].status" value="取消 " @click="Cancle(0,i)"/>
          <button class="hide_reply"  @click="ShowInput(0,i)">回复</button>
          <img class="thumb_up " v-if="!root_com.is_love" src="../../static/images/good_none.png" @click="createPoint(root_com.id,1)" onmouseover="this.src='../../static/images/good_exist.png'" onmouseout="this.src='../../static/images/good_none.png'" >
          <img class="thumb_up " v-if="root_com.is_love" src="../../static/images/good_exist.png" >
          {{root_com.point_love_nums}}

          <img class="thumb_up " src="../../static/images/msg_none.png" @click="ShowChildCommend(i)" onmouseover="this.src='../../static/images/msg_exist.png'" onmouseout="this.src='../../static/images/msg_none.png'">
          {{root_com.child_com.length}}
        </div>
        <div class="child_commend" v-if="show[i].status">
          <div class="child_commend_reply" v-for="(child,j) in root_com.child_com" :key="j">
            <div class="commend_up">
              <img class="root_commend_icon" :src="child.from_uid.user_profile.image">
              <div class="root_commend_name">
                <span >{{child.from_uid.user_profile.nick_name}}</span>
                <span class="reply_charecter">&nbsp;&nbsp;回复&nbsp;&nbsp;</span>
                <span>{{child.to_uid.user_profile.nick_name}}</span>
              </div>
            </div>
            <div class="commend_content">
              {{child.comment}}
            </div>
            <div class="commend_down">
              <input type="text" class="reply_input"  v-show="show_input[i+1][j].status" v-model="show_input[i+1][j].value">
              <input type="button" class="reply_button" v-show="show_input[i+1][j].status" value="评论" @click="Reply(root_com.id, child.id, child.from_uid.id, 2,i+1, j)"/>
              <input type="button" class="reply_cancle"  v-show="show_input[i+1][j].status" value="取消 " @click="Cancle(i+1,j)"/>
              <button class="hide_reply" @click="ShowInput(i+1,j)">回复</button>
              <img class="thumb_up msg" v-if="!child.is_love" src="../../static/images/good_none.png" @click="createPoint(child.id, 2)" onmouseover="this.src='../../static/images/good_exist.png'"  onmouseout="this.src='../../static/images/good_none.png'">
              <img class="thumb_up msg" v-if="child.is_love " src="../../static/images/good_exist.png">
              {{child.point_love_nums}}
            </div>
          </div>
        </div>

      </div>
    </div>

  </div>

</template>

<script>
  import 'video.js/dist/video-js.css'
  import { videoPlayer } from 'vue-video-player'
  import {mapGetters} from 'vuex'
  // Similarly, you can also introduce the plugin resource pack you want to use within the component
  // import 'some-videojs-plugin'
  export default {
    components: {
      videoPlayer
    },
    data() {
      return {
        videoInfo:{},
        playerOptions: {

          // videojs options
          muted: false,
          language: 'zh-CN',
          preload: 'auto',
          loop: false,
          aspectRatio: '16:9',
          fluid: true,
          playbackRates: [0.7, 1.0, 1.5, 2.0],
          sources: [{
            type: "video/mp4",
            src: ""
          }],
          poster: "",
        },
        show:[
        ],
        show_input: [
            [

            ]
        ],
//        input_value:{
//            "s0":'',
//          "s01":'',
//          "s02":''
//        },
        a:{},
        videoRecommend:{},
        createRootCommend:'',
        createChildCommend:'',
        is_i:''
      }
    },
    mounted:function () {
      let id = this.videoId.toString()
//      获取视频资源
      this.$axios.get('video/'+id+'/')
        .then((response)=>{
          this.videoInfo = response.data;
          this.playerOptions.sources[0].src = response.data.url
          this.playerOptions.poster = response.data.video_img
          console.log(this.videoInfo)
        })
        .catch(function (err) {
          console.log(err )
        })
//      获取推荐的视频列表
      this.$axios.get('video_recommend/'+id + '/')
        .then((response)=>{
          this.videoRecommend = response.data

        })
        .catch(function (err) {
          console.log(err)
        })
    this.getComment(id);

    },
    computed: {
      ...mapGetters({
        videoId:'getVideoId'
      }),
      player() {
        return this.$refs.videoPlayer.player
      }
    },
    methods: {
      // listen event
      onPlayerPlay(player) {
        console.log('player play!', player)
      },
      onPlayerPause(player) {
        console.log('player pause!', player)
      },
      // ...player event
      // or listen state event
      playerStateChanged(playerCurrentState) {
        console.log('player current update state', playerCurrentState)
      },
      // player is ready
      playerReadied(player) {
        console.log('the player is readied', player)
//         you can use it to do something...
        // player.[methods]
      },

//      子评论的显示和隐藏
      ShowChildCommend(num){
        if( this.show[num].status === false) {
          this.show[num].status = true
        }
        else {
          this.show[num].status = false
        }
      },
//      获取推荐视频
      RecommendVideo(type){

        this.$store.dispatch('changeVideoId',type)
        let id = this.videoId.toString()
        this.$axios.get('video/'+id+'/')
          .then((response)=>{
            this.videoInfo = response.data;
            this.playerOptions.sources[0].src = response.data.url
            this.playerOptions.poster = response.data.video_img
            console.log(this.videoInfo)
          })
          .catch(function (err) {
            console.log(err )
          })
        this.$axios.get('/video_recommend/'+id + '/')
          .then((response)=>{
            this.videoRecommend = response.data
          })
          .catch(function (err) {
            console.log(err)
          })
      },
//      创建跟评论
      CreateRootCommend(){
          if(this.createRootCommend !== ''){
           let video = this.videoId
           let  comment = this.createRootCommend
            this.$axios.post('video_com/',{
              video,
              comment
              })
              .then((response)=>{
                this.getComment(video);
                alert("评论成功")
              })
              .catch(function (err) {
                console.log(err)
                alert("不知道啥子离家出走了，反正上传不了评论了╥﹏╥...")
              })
          }
      },
      CreateChildCommend() {
        if(this.createChildCommend != ''){
          let video = this.videoId
          this.$axios.post('/video_reply/',{
            'comment_id':this.root_com.id,
            'reply_id': this.root_com.video,
            'to_uid': this.root_com.user.id,
            'comment':this.createChildCommend
          })
            .then((response)=>{
              this.getComment(video);
              alert("评论成功")
            })
            .catch(function (err) {
              console.log(err)
              alert("不知道啥子离家出走了，反正上传不了评论了╥﹏╥...")
            })
        }
      },
//      评论输入框的显示和隐藏
      ShowInput(i,j){
//          if( this.show[i].status === false) {
//            this.show[i].status = true
//          }
//          else {
//            this.show[i].status = false
//          }
          if(this.show_input[i][j].status === true){
            this.show_input[i][j].status = false;
          }
         else{
            this.show_input[i][j].status = true;
          }
      },
//      跟评论和自评论的回复
      Reply(comment_id, reply_id, to_uid, reply_type ,i, j){

          let comment = this.show_input[i][j].value;
          console.log(comment_id, reply_id, to_uid, reply_type, comment);
          this.$axios.post('/video_reply/',{
            comment_id: comment_id,
            reply_id: reply_id,
            to_uid: to_uid,
            reply_type: reply_type,
            comment: comment
          })
            .then((res)=>{
              this.show_input[i][j].value = ''
              this.getComment(this.videoId)
            })
      },
//      取消回复
      Cancle(i,j){
//          var pos = 's'+index.toString()
          this.show_input[i][j].status = false;
          this.show_input[i][j].value = ''
      },

      createPoint(id, type) {
          console.log(id, type)
          this.$axios.post('point/',{
            video_comment: id,
            reply_type: type
          })
            .then((response)=>{
              alert("点赞成功1")
              this.getComment(this.videoId);
            })
            .catch(function (err) {
              console.log(err)
            })
      },
      getComment(id){
        var that=this;
//      获取视频评论信息
        this.$axios.get('video_com/'+id +'/')
          .then((response)=>{
            this.a = response.data;

            if(this.a.length >0){
              for(let i=0;i<this.a.length;i++){
                that.show_input[0].push({
                  'status':false,
                  'value':''
                });
               that.show.push({status:false})
               that.show_input.push([]);
               for(var j=0;j<that.a[i].child_com.length;j++){
                 that.show_input[i+1].push({status:false,value:''});
               }
              }
            }
          })
          .catch(function (err) {
            console.log(err)
          })
      }
    }
  }
</script>


<style scoped>
  .con{width: 100%;height: auto;margin-top: 5px;padding: 0;background-color: #f5f5f5;}
  .video{width:100%;height: 770px;padding-top: 20px;}
  .left{width: 1190px;margin-left: 100px;height: 750px;float: left;box-shadow: 0 0 3px #d1d1d1;background-color: #ffffff;margin-right: 10px;}
  .title{width: 1150px;height: 100px;padding-left: 50px;}
  .name{font-size: 30px;color: #00BC9B;}
  .desc{font-size: 25px;color: #636363;}
  .click_num{font-size: 27px;color: #d7d7d7;}
  .right{width: 300px;height: 720px;float: right;margin-right: 80px;box-shadow: 0 0 3px #d1d1d1;background-color: #ffffff;
  }
  .recommend{width:95%;margin-left: 5%;height: 110px;margin-bottom: 10px;}
  .rec_img{width: 140px;height: 100px;margin-top: 10px;float: left;}
  img{width: inherit;height: inherit;border-radius: 10px;}
  .content{float: right;height: 100px;width: 140px;margin-top: 10px;}
  .content_title{width: 100%;height: 70px;font-size: 18px;color: #595959  }
  .content_title:hover{color: #00BC9B}
  .con_click_num{width: 100%;height:20px;}
  .rec_click_img{width: 30px;height: 100%;border-radius: 1px;}
  .rec_click_num{font-size: 18px;color: #8f8f8f;overflow: hidden;}

  /*.root_commend:hover .child_commend{display: inline-block;}*/
  /*评论功能的css*/
  /*用户自己的根评论你*/
  .create_root_commend{height: 50px;line-height: 50px;border-radius: 10px;width: 800px;margin-left: 110px;border: none;box-shadow: 0 0 3px #5b5b5b;padding-left: 20px;font-size: 21px;margin-bottom: 20px;margin-top: 10px;}
  .create_root_button{height: 50px;width: 70px;background-color:#00BC9B; border: none;font-size: 20px;color: white;margin-left: 15px;border-radius: 5px;cursor:pointer;}
  /*展示评论*/
  .commend{width: 1190px;margin-left: 100px;height: auto;box-shadow: 0 0 3px #d1d1d1;background-color: #ffffff;margin-top: 20px;padding-top: 20px; }
  .root_commend{width: 1000px;height: auto;display: inline-block;margin-left: 100px;border-bottom: 1px solid  #d1d1d1;margin-top: 20px;}
  .child_commend{
    width: 950px;
    height:auto;
    margin-left: 10px;
    margin-bottom: 10px;
    background-color: #f7f8fa;
    margin-top: 50px;
  }
  .commend_up{width: 100%;height: 40px;line-height: 40px;}
  .root_commend_icon{width: 40px;height: 40px;border-radius: 20px;display: inline-block;float: left;line-height: 40px;margin-left:10px; }
  .root_commend_name{ height: 40px;  margin-left: 60px; vertical-align: middle;  display: table-cell;width: 970px;font-size: 20px;padding-left: 10px; }
  .root_commend_time{height: 40px; vertical-align: middle;  display: table-cell;float: right; width:300px;text-align: center;color: grey;}
  .commend_content{width: 95%;margin-left: 20px;height: auto;margin-top: 10px;line-height: 30px;font-size: 19px;}
  .commend_down{width: auto;height:50px;line-height:50px; display: table-cell;float: right;color: grey; margin-right: 40px;margin-top: 10px;}
  .thumb_up{width: 23px;height: 23px;vertical-align: middle; margin-left: 15px;margin-right: 5px; cursor:pointer;}
  .hide_reply{margin-left: 20px;background-color:#00BC9B; border: none;font-size: 15px;color: white;margin-left: 15px;border-radius: 5px;cursor:pointer; }
  .child_commend_reply{width: 95%;margin-left: 20px;height: auto;  border-bottom: solid 1px #d1d1d1;display: inline-block;margin-top: 20px;}
  .reply_charecter{color: #7a7a7a;}
  .reply_input{height: 30px; width: 400px;vertical-align: middle;display: table-cell;border-radius: 10px;border: none;box-shadow: 0 0 3px grey;padding-left:7px;font-size: 19px;}
  .reply_button{border: none;background-color: #00BC9B;width: 40px;height: 28px;border-radius: 5px;color: white;font-size: medium;vertical-align: middle;display: table-cell;margin-left: 10px;padding-right: 3px;}
  .reply_cancle{border: none;background-color: #c1c1c1;width: 40px;height: 28px;border-radius: 5px;color: white;font-size: medium;vertical-align: middle;display: table-cell;margin-left: 5px;padding-left: 3px;}
</style>

