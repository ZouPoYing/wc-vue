<template>
    <div id="contact">
        <section>
            <div class="weui-cells_contact-head weui-cells weui-cells_access" style="margin-top:-1px">
                <router-link to="/contact/new-friends" class="weui-cell">
                    <div class="weui-cell_hd"> <img class="img-obj-cover" src="../../assets/images/contact_top-friend-notify.png"> </div>
                    <div class="weui-cell_bd weui-cell_primary">
                        <p>新的朋友</p>
                    </div>
                </router-link>
            </div>
            <template v-for="(item,key) in info">
                <div class="weui-cells">
                    <router-link :key="item.wxid" :to="{path:'/contact/details',query:{wxid:item.userId}}" class="weui-cell weui-cell_access" tag="div">
                        <div class="weui-cell__hd">
                            <img :src="item.userHead" class="home__mini-avatar___1nSrW">
                        </div>
                        <div class="weui-cell__bd">
                            {{item.userName}}
                        </div>
                    </router-link>
                </div>
            </template>
</section>
</div>
</template>
<script>
    import axios from "axios";

    export default {
        mixins: [window.mixin],
        data() {
            return {
                "pageName": "通讯录",
              info: []
            }
        },
      created() {
          this.initPage()
      },
      methods: {
          getFriend() {
            var that = this
            axios.post('/friend/getFriend',{
              userId: that.$store.state.user.userId
            }).then(function (res) {
              if (res.data.success) {
                that.info = res.data.friends
                that.$store.commit('setFriends',res.data.friends);
              }
            })
          },
          initPage() {
            this.getFriend()
          }
      },
      mounted() {
        // mutations.js中有介绍
        this.$store.commit("toggleTipsStatus", -1)
        this.timer = setInterval(this.getFriend, 1000);
      },
      beforeDestroy() {
        clearInterval(this.timer);
      },
        activated() {
            this.$store.commit("toggleTipsStatus", -1)
        },
        computed: {
            // 从好友中提取出首字母 再排序 算法一般
            initialList: function() {
                var initialList = [],
                    users = this.$store.state.allContacts,
                    max = users.length
                for (var i = 0; i < max; i++) {
                    if (initialList.indexOf(users[i].initial.toUpperCase()) == -1) {
                        initialList.push(users[i].initial.toUpperCase())
                    }
                }
                return initialList.sort()
            },
            // 将好友根据首字母进行分类 算法一般
            contactsList: function() {
                var contactsList = {},
                    users = this.$store.state.allContacts,
                    max = users.length;
                for (var i = 0; i < this.initialList.length; i++) {
                    var protoTypeName = this.initialList[i]
                    contactsList[protoTypeName] = []
                    for (var j = 0; j < max; j++) {
                        if (users[j].initial.toUpperCase() === protoTypeName) {
                            contactsList[protoTypeName].push(users[j])
                        }
                    }
                }
                return contactsList
            }
        }
    }
</script>
<style>
    @import "../../assets/css/contact.css";
</style>
