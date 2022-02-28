<template>
<!--个人信息组件-->
    <div class="profile">
        <header id="wx-header">
            <div class="center">
                <router-link to="/self" tag="div" class="iconfont icon-return-arrow">
                    <span>我</span>
                </router-link>
                <span>个人信息</span>
            </div>
        </header>
        <div class="weui-cells">
            <div class="weui-cell" id="avatarCell">
                <div class="weui-cell__bd">
                    <p>头像</p>
                </div>
                <el-upload class="avatar-uploader"
                           action
                           :http-request="handleUploadForm"
                           :show-file-list="false"
                           :before-upload="beforeAvatarUpload">
                    <div class="weui-cell__ft">
                        <img :src="$store.state.user.userHead" style="width: 50px;height: 50px;border-radius: 4px;">
                    </div>
                </el-upload>
            </div>
            <div class="weui-cell" @click="pop('名字')">
                <div class="weui-cell__bd">
                    <p>名字</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.userName}}
                </div>
            </div>
            <div class="weui-cell">
                <div class="weui-cell__bd">
                    <p>微信号</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.userId}}
                </div>
            </div>
            <router-link to="/self/profile/my-qrcode" class="weui-cell weui-cell_access">
                <div class="weui-cell__bd">
                    <p>我的二维码</p>
                </div>
                <div class="weui-cell__ft">
                    <img src="../../assets/images/contact_add-friend-my-qr.png" style="vertical-align: middle;;width:24px" class="_align-middle">
                </div>
            </router-link>
        </div>

        <div class="weui-cells">
            <div class="weui-cell" @click="pop('性别')">
                <div class="weui-cell__bd">
                    <p>性别</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.sex}}
                </div>
            </div>
            <div class="weui-cell" @click="pop('省')">
                <div class="weui-cell__bd">
                    <p>省</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.province}}
                </div>
            </div>
            <div class="weui-cell" @click="pop('市')">
                <div class="weui-cell__bd">
                    <p>市</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.city}}
                </div>
            </div>
            <div class="weui-cell"  @click="pop('个性签名')">
                <div class="weui-cell__bd">
                    <p>个性签名</p>
                </div>
                <div class="weui-cell__ft">
                    {{$store.state.user.signature}}
                </div>
            </div>
        </div>
        <el-dialog class="dialog" width="80%" :title="title" :visible.sync="dialogFormVisible" append-to-body>
            <el-form :model="form">
                <el-form-item>
                    <el-input v-model="form.msg" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="success" @click="change">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import axios from 'axios'
    export default {
        data() {
            return {
                pageName: "个人信息",
                dialogFormVisible: false,
                title:'',
                form: {
                  msg: ''
                }
            }
        },
      created() {
      },
      methods: {
        handleUploadForm(param) {
          var that = this
          let formData = new FormData();
          formData.append('userId', this.$store.state.user.userId)
          formData.append('file', param.file)
          axios.post('file/upload',formData).then(function(res){
            that.head(res.data.fileId);
          })
        },
        beforeAvatarUpload(file) {
          const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
          const isLt2M = file.size / 1024 / 1024 < 2;
          if (!isJPG) {
            this.$toast('上传头像图片只能是 JPG 格式!');
          }
          if (!isLt2M) {
            this.$toast('上传头像图片大小不能超过 2MB!');
          }
          return isJPG && isLt2M;
        },
          head(fileId) {
            var that = this
            axios.post('/user/head', {
              userId: that.$store.state.user.userId,
              fileId: fileId
            }).then(function(res) {
              if (res.data.success) {
                that.$store.commit('setUser', res.data.user);
              }
            })
          },
          pop(v) {
            this.title = v
            this.dialogFormVisible = true
          },
          change() {
            var that = this
            axios.post('/user/change', {
              type: that.title,
              userId: that.$store.state.user.userId,
              msg: that.form.msg
            }).then(function(res){
              if (res.data.success) {
                that.title = ''
                that.form.msg = ''
                that.dialogFormVisible = false
                that.$store.commit('setUser',res.data.user);
              } else {
                that.dialogFormVisible = false
                that.$toast(res.data.msg)
              }
            })
          }
        }
    }
</script>

<style scoped>

</style>
