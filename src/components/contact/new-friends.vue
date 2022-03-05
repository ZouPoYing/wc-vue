<template>
    <div :class="{'search-open-contact':!$store.state.headerStatus}">
        <header id="wx-header">
            <div class="other"><span>添加朋友</span></div>
            <div class="center">
                <router-link to="/contact" tag="div" class="iconfont icon-return-arrow">
                    <span>通讯录</span>
                </router-link>
                <span>新的朋友</span>
            </div>
        </header>
         <!--这里的 search 组件的样式也需要修改一下-->
        <search></search>
        <div class="weui-cells message-list" v-for="(item, index) in info"  v-bind:key="index">
            <div class="weui-cell">
                <div class="weui-cell__hd">
                    <img :src="item.head" alt=""></div>
                <div class="weui-cell__bd weui-cell_primary">
                    <p><b>{{item.applyName}}</b></p>
                    <p><span>{{item.tips}}</span></p>
                </div>
                <div v-if="item.status == '好友'" class="weui-cell__ft">已添加</div>
                <div v-else-if="item.status == '拒绝'" class="weui-cell_ft">已拒绝</div>
                <div v-else-if="item.status == '申请'" class="weui-cell_ft">
                    <a href="javascript:;" @click.stop="accept(item.friendId)" class="weui-btn weui-btn_mini weui-btn_primary">接受</a>
                    <a href="javascript:;" @click.stop="refuse(item.friendId)" class="weui-btn weui-btn_mini weui-btn_primary">拒绝</a>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import search from "../common/search"
    import axios from "axios";
    export default {
      data() {
        return {
          pageName: "新的朋友",
          info: []
        }
      },
      created() {
        this.initPage()
      },
      mounted() {
        this.timer = setInterval(this.getApply, 1000);
      },
      beforeDestroy() {
        clearInterval(this.timer);
      },
      methods: {
        refuse(friendId) {
          var that = this
          axios.post('/friend/refuseByFriendId',{
            friendId: friendId
          }).then(function (res) {
            if (res.data.success) {
              that.initPage()
            }
          })
        },
        accept(friendId) {
          var that = this
          axios.post('/friend/acceptByFriendId',{
            friendId: friendId
          }).then(function (res) {
            if (res.data.success) {
              that.initPage()
            }
          })
        },
        getApply() {
          var that = this
          axios.post('/friend/getApply',{
            userId: this.$store.state.user.userId
          }).then(function (res) {
            if (res.data.success) {
              that.info = res.data.apply
            }
          })
        },
        initPage() {
          this.getApply()
        }
      },
        components: {
            search
        }
    }
</script>
<style>
    @import "../../assets/css/new-friends.css";
</style>
