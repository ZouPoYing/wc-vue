<template>
    <!--微信组件-->
  <div id="wechat">
    <ul class="wechat-list">
        <msg-item v-for="(item,index) in info" :item="item" class="list-row line-bottom" :key="index"></msg-item>
    </ul>
  </div>
</template>
<script>
    import search from "../common/search"
    import msgItem from "../wechat/msg-item"
    import axios from "axios";
    export default {
        components: {
            search,
            msgItem
        },
        mixins: [window.mixin],
        data() {
            return {
              info: [],
                "pageName": "微信"
            }
        },
      methods: {
          getChatRoom() {
            var that = this
            axios.post('/message/getChatRoom',{
              userId: that.$store.state.user.userId
            }).then(function (res) {
              if (res.data.success) {
                that.info = res.data.chatRoom
              }
            })
          },
          initPage() {
            this.getChatRoom()
          }
      },
      created() {
          this.initPage()
      }
    }
</script>
<style>
    @import "../../assets/css/wechat.css";
</style>