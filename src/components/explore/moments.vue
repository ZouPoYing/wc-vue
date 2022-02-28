<template>
    <div id="moments">
        <header id="wx-header">
            <div class="center">
                <router-link to="/explore" tag="div" class="iconfont icon-return-arrow">
                    <span>发现</span>
                </router-link>
                <span>朋友圈</span>
                <img width="25px" @click="show" style="top: 10px;position:relative;left: 120px;" src="../../assets/images/sendCircle.png">
            </div>
        </header>
        <div class="home-pic" :style="{background: 'url(' + $store.state.user.back + ') no-repeat center center'}">
            <div class="home-pic-base">
                <div class="top-pic">
                    <div class="top-pic-inner">
                        <img :src="$store.state.user.userHead">
                    </div>
                </div>
                <div class="top-name _ellipsis">{{$store.state.user.userName}}</div>
            </div>
        </div>
        <el-upload class="avatar-uploader" id="clickBack" ref="clickBack" action :http-request="backHandleUploadForm"
                   :show-file-list="false" :before-upload="beforeAvatarUpload">
            <img width="25px" style="position:relative;left: 20px;" src="../../assets/images/changeback.png">
        </el-upload>
        <el-dialog class="dialog" width="80%" title="发朋友圈" :visible.sync="dialogFormVisible" append-to-body>
            <el-form>
                <el-form-item>
                    <el-input type="textarea" maxlength="200" autosize show-word-limit v-model="msg" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-upload :limit="9" action :http-request="handleUploadForm"
                            list-type="picture-card"
                            :file-list="fileList"
                            :on-success="fileSucces"
                            :before-upload="beforeAvatarUpload"
                            :on-remove="handleRemove">
                        <i class="el-icon-plus" style="width: 33%"></i>
                    </el-upload>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="success" @click="sendCircle">确 定</el-button>
            </div>
        </el-dialog>
        <el-dialog class="dialog" width="80%" title="评论" :visible.sync="PLdialogFormVisible" append-to-body>
            <el-form>
                <el-form-item>
                    <el-input type="textarea" maxlength="50" autosize show-word-limit v-model="text" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="PLdialogFormVisible = false">取 消</el-button>
                <el-button type="success" @click="sendComment">确 定</el-button>
            </div>
        </el-dialog>
        <div v-for="(item,index) in info" :key="index">
            <div class="weui-cell moments__post">
                <div class="weui-cell__hd">
                    <img :src="item.userHead">
                </div>
                <div class="weui-cell__bd" style="width: 300px">
                    <a class="title">
                        <span>{{item.userName}}</span>
                    </a>
                    <p id="paragraph" class="paragraph">{{item.text}}</p>
                    <!-- 相册 -->
                    <div class="thumbnails">
                        <div class="thumbnail" v-for="(pic,index) in item.pics" :key="index">
                            <img :src="pic">
                        </div>
                    </div>
                    <!-- 资料条 -->
                    <div class="toolbar">
                        <p class="timestamp">{{item.time}}</p>
                        <div>
                            <div id="actionMenu" class="actionMenu slideIn">
                                <p class="actionBtn" id="btnLike"><i class="icon icon-96"></i></p>
                                <p class="actionBtn" id="btnComment"><i class="icon icon-3"></i></p>
                            </div>
                        </div>
                        <span class="likeActionToggle" @click="like(item.circleId)">
                            <i class="el-icon-thumb"></i>
                        </span>
                        <span class="commentActionToggle" @click="comment(item.circleId)">
                            <i class="el-icon-chat-square"></i>
                        </span>
                    </div>
                    <!-- 赞／评论区 -->
                    <div class="liketext">
                        <i class="icon icon-96"></i>
                        <div v-for="(like,index) in item.likes" :key="index">
                            <span class="nickname" v-if="index != 0">,</span>
                            <span class="nickname">{{like}}</span>
                        </div>
                        <div style="width:100%">
                        </div>
                        <div v-for="(comment,index) in item.comments" :key="index">
                            <div style="width:100%">
                            </div>
                            <div>
                                <span class="nickname">{{comment.userName}}：</span>
                                <span>{{comment.text}}</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import axios from "axios";

    export default {
      data() {
        return {
          pictureList: [],
          fileList: [],
          info: [],
          msg: '',
          dialogFormVisible: false,
          PLdialogFormVisible: false,
          text: '',
          circleId: ''
        }
      },
      methods: {
        comment(circleId) {
          this.circleId = circleId
          this.PLdialogFormVisible = !this.PLdialogFormVisible
        },
        sendComment() {
          var that = this
          axios.post('/comment/comment', {
            userId: that.$store.state.user.userId,
            circleId: that.circleId,
            text: that.text
          }).then(function(res){
            if (res.data.success) {
              that.$toast('操作成功')
              that.initPage()
              that.circleId = ''
              that.text = ''
              that.PLdialogFormVisible = !that.PLdialogFormVisible
            } else {
              that.$toast(res.data.msg)
            }
          })
        },
        like(circleId) {
          var that = this
          axios.post('/likes/likes', {
            userId: that.$store.state.user.userId,
            circleId: circleId
          }).then(function(res){
            if (res.data.success) {
              that.$toast('操作成功')
              that.initPage()
            } else {
              that.$toast(res.data.msg)
            }
          })
        },
        getCircle() {
          var that = this
          axios.post('/circle/getCircle', {
            userId: that.$store.state.user.userId,
          }).then(function(res){
            if (res.data.success) {
              that.info = res.data.circle
            }
          })
        },
        sendCircle() {
          var that = this
          axios.post('/circle/sendCircle', {
            userId: that.$store.state.user.userId,
            text: that.msg,
            pic: that.fileList + []
          }).then(function(res){
            if (res.data.success) {
              that.msg = ''
              that.pictureList = []
              that.fileList = []
              that.$toast('操作成功')
              that.initPage()
            } else {
              that.$toast(res.data.msg)
            }
            that.show()
          })
        },
        back(fileId) {
          var that = this
          axios.post('/user/back', {
            userId: that.$store.state.user.userId,
            fileId: fileId
          }).then(function(res) {
            if (res.data.success) {
              that.$store.commit('setUser', res.data.user);
            }
          })
        },
        backHandleUploadForm(param) {
          var that = this
          let formData = new FormData();
          formData.append('userId', this.$store.state.user.userId)
          formData.append('file', param.file)
          axios.post('file/upload',formData).then(function(res){
            that.back(res.data.fileId);
          })
        },
        handleUploadForm(param) {
          var that = this
          let formData = new FormData();
          formData.append('userId', this.$store.state.user.userId)
          formData.append('file', param.file)
          axios.post('file/upload',formData).then(function(res){
            that.fileList.push(res.data.fileId);
            that.pictureList.push(param.file.uid)
          })
        },
        beforeAvatarUpload(file) {
          const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
          const isLt2M = file.size / 1024 / 1024 < 2;
          if (!isJPG) {
            this.show()
            this.$toast('上传头像图片只能是 JPG 格式!');
          }
          if (!isLt2M) {
            this.show()
            this.$toast('上传头像图片大小不能超过 2MB!');
          }
          return isJPG && isLt2M;
        },
        handleRemove(file) {
          const uid = file.uid;
          const i = this.pictureList.findIndex((x) => x === uid);
          this.pictureList.splice(i, 1);
          this.fileList.splice(i, 1);
        },
        show() {
            this.dialogFormVisible = !this.dialogFormVisible
        },
        initPage() {
            this.getCircle()
        }
      },
      created() {
        this.initPage()
      }
    }
</script>
<style>
    @import "../../assets/css/moments.css";

    .el-icon-picture{
        width: 50px;
        height: 50px;
    }

    .el-upload--picture-card {
        width: 70px;
        height: 70px;
    }
    .el-upload-list__item {
        width: 70px ;
        height: 70px ;
    }
    .el-upload-list__item-actions {
        width: 70px ;
        height: 70px ;
    }
    .is-ready {
        width: 70px ;
        height: 70px ;
    }
    .el-icon-plus {
        position: relative;
        bottom: 25px;
    }
    #moments
    .moments__post
    .likeActionToggle{
        width:25px;
        height:25px;
        margin-left:auto;
        border-radius:1px;
        position:absolute;
        right:25px;
        display:-webkit-box;
        display:-webkit-flex;
        display:-ms-flexbox;
        display:flex;
        -webkit-box-align:center;
        -webkit-align-items:center;
        -ms-flex-align:center;
        align-items:center;
        -webkit-box-pack:center;
        -webkit-justify-content:center;
        -ms-flex-pack:center;
        justify-content:center
    }
    #moments
    .moments__post
    .commentActionToggle{
        width:25px;
        height:25px;
        margin-left:auto;
        border-radius:1px;
        position:absolute;
        right:0px;
        display:-webkit-box;
        display:-webkit-flex;
        display:-ms-flexbox;
        display:flex;
        -webkit-box-align:center;
        -webkit-align-items:center;
        -ms-flex-align:center;
        align-items:center;
        -webkit-box-pack:center;
        -webkit-justify-content:center;
        -ms-flex-pack:center;
        justify-content:center
    }
</style>