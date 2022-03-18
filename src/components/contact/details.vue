<template>
    <div class="profile">
        <header id="wx-header">
            <div class="center">
                <router-link to="/contact" tag="div" class="iconfont icon-return-arrow">
                    <span>通讯录</span>
                </router-link>
                <span>详细资料</span>
            </div>
        </header>
        <div class="weui-cells">
            <div class="weui-cell">
                <div class="weui-cell__hd"><img :src="info.userHead" alt="" class="self-header" style="width:60px"></div>
                <div class="weui-cell__bd">
                    <h4 class="self-nickname">{{info.userName}}<span class="gender" :class="[info.sex== '男'?'gender-male':info.sex == '女'?'gender-female':'']"></span></h4>
                    <p class="self-wxid" style="font-size: 13px;color: #999;">微信号: {{info.userId}}</p>
                </div>
            </div>
        </div>
        <div class="weui-cells">
            <div class="weui-cell weui-cell_access">
                <div class="weui-cell__bd">
                    <p>设置备注和标签</p>
                </div>
                <div class="weui-cell__ft">

                </div>
            </div>
        </div>
        <div class="weui-cells">
            <div class="weui-cell">
                <div class="weui-cell__bd">
                    <p>地区</p>
                </div>
                <div class="weui-cell__ft" style="-webkit-flex: 4;text-align: left;">
                    <span>{{info.province}}  {{info.city}}</span>
                </div>
            </div>
            <div class="weui-cell">
                <div class="weui-cell__bd">
                    <p>个性签名</p>
                </div>
                <div class="weui-cell__ft" style="-webkit-flex: 4;text-align: left;">
                    <span>{{info.signature}}</span>
                </div>
            </div>
            <div class="weui-cell weui-cell_access">
                <div class="weui-cell__bd">
                    <p>个人相册</p>
                </div>
                <div class="weui-cell__ft" style="-webkit-flex: 4;text-align: left;">
                    <div class="album-list">
                        <img :src="item.imgSrc" style="width:50px;margin:0 5px" v-for="item in userInfo.album">
                    </div>
                </div>
            </div>
            <div class="weui-cell weui-cell_access">
                <div class="weui-cell__bd">
                    <p>更多</p>
                </div>
                <div class="weui-cell__ft">

                </div>
            </div>
        </div>
        <a v-if="isFriend" href="javascript:;" @click="goto" class="weui-btn weui-btn_primary" style="width:90%;margin-top:20px;">发消息</a>
        <a v-else href="javascript:;" class="weui-btn weui-btn_primary" @click="showDia" style="width:90%;margin-top:20px;">加好友</a>
        <el-dialog class="dialog" width="80%" title="申请添加朋友" :visible.sync="dialogFormVisible" append-to-body>
            <el-form>
                <el-form-item>
                    <el-input type="textarea" maxlength="200" autosize show-word-limit v-model="tips" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="success" @click="addFriend">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import axios from "axios";
    export default {
        data() {
            return {
                pageName: "",
              info: {},
              userInfo: {},
              isFriend: false,
              dialogFormVisible: false,
              tips: ''
            }
        },
      created() {
          this.initPage()
      },
      methods: {
        goto() {
          this.$router.push({
            path: '/wechat/dialogue',
            query: {
              name: this.info.userName,
              friend: this.info.userId,
              isFriend: true
            }
          })
        },
        initPage() {
          this.getUserInfo()
          this.isFriendF()
          this.setTips()
        },
        addFriend() {
          var that = this
          axios.post('/friend/friendApply', {
            userId: this.$store.state.user.userId,
            friend: this.$route.query.wxid,
            tips: this.tips
          }).then(function(res){
            if (res.data.success) {
              that.$toast('申请成功');
            } else {
              that.$toast(res.data.msg);
            }
            that.showDia()
          })
        },
        getUserInfo() {
          var that = this
          axios.post('/user/getMyInfoByUserId', {
            userId: this.$route.query.wxid
          }).then(function(res){
            if (res.data.success) {
              that.info = res.data.user
            } else {
              that.$toast(res.data.msg);
            }
          })
        },
        isFriendF() {
          var that = this
          axios.post('/friend/isFriend', {
            userId: this.$store.state.user.userId,
            friend: this.$route.query.wxid
          }).then(function(res){
            if (res.data.success) {
              that.isFriend = res.data.isFriend
            }
          })
        },
        showDia() {
          this.dialogFormVisible = !this.dialogFormVisible
        },
        setTips() {
          this.tips = '我是' + this.$store.state.user.userName
        }
      },
      computed: {

      }
    }
</script>
<style>

</style>
