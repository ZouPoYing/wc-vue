<template>
  <!--我 组件-->
  <div id="self">
    <div class="weui-tab__content" style="display: block;">
      <div class="weui-cells">
        <router-link to="/self/profile" class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img :src="info.userHead" alt="" class="self-header">
          </div>
          <div class="weui-cell__bd">
            <h4 class="self-nickname">{{info.userName}}</h4>

            <p class="self-wxid">微信号: {{info.userId}}</p>
          </div>
          <div class="weui-cell__ft">
            <img src="../../assets/images/chat-info-qr.png">
          </div>
        </router-link>
      </div>
      <div class="weui-cells">
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_more-my-album.png">
          </div>
          <div class="weui-cell__bd">
            <p>相册</p>
          </div>
        </div>
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_more-my-favorites.png">
          </div>
          <div class="weui-cell__bd">
            <p>收藏</p>
          </div>
        </div>
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_more-my-bank-card.png">
          </div>
          <div class="weui-cell__bd">
            <p>钱包</p>
          </div>
        </div>
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_my-card-package-icon.png">
          </div>
          <div class="weui-cell__bd">
            <p>卡券</p>
          </div>
        </div>
      </div>
      <div class="weui-cells">
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_more-expression.png">
          </div>
          <div class="weui-cell__bd">
            <p>表情</p>
          </div>
        </div>
      </div>
      <div class="weui-cells">
        <div class="weui-cell weui-cell_access">
          <div class="weui-cell__hd">
            <img src="../../assets/images/me_more-setting.png">
          </div>
          <div class="weui-cell__bd">
            <p>设置</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
    import axios from "axios";

    export default {
        mixins: [window.mixin],
        data() {
            return {
                "pageName": "我",
              info: {}
            }
        },
        created() {
            this.getMyInfo()
        },
        methods: {
          getMyInfo() {
            var that = this
            axios.post('/user/getMyInfoByUserId', {
              userId: this.$store.state.user.userId
            }).then(function(res){
              if (res.data.success) {
                that.info = res.data.user
              } else {
                that.$toast(res.data.msg);
              }
            })
          }
        },
        mounted() {
            this.$store.commit("toggleTipsStatus", -1)
        },
        activated() {
            this.$store.commit("toggleTipsStatus", -1)
        }
    }
</script>
<style>
    @import "../../assets/css/self.css";
</style>