<template>
    <div>
        <header id="wx-header">
            <div class="other">
                <!--群详情 -->
                <span class="iconfont icon-chat-friends" @click="goto" v-show="$route.path==='/wechat/dialogue' && $route.query.isFriend===false"></span>
            </div>
            <div class="center">
                <router-link to="/" tag="div" class="iconfont icon-return-arrow">
                    <span>微信</span>
                </router-link>
                <span>{{pageName}}</span>
                <span class="parentheses"
                      v-show='$route.query.group_num&&$route.query.group_num!=1'>{{$route.query.group_num}}</span>
            </div>
        </header>
        <section class="dialogue-section clearfix">
            <div class="row clearfix" v-for="(item,index) in info">
                <div v-if="item.mark == 'to'">
                    <img :src="item.userHead" class="header">
                    <div v-if="item.type == 'friend'">
                        <div class="left-name">{{item.userName}}</div>
                        <router-link :to="{path:'/contact/details',query:{wxid:item.value}}" tag="div">
                            <p class="text" v-more>
                                <img style="width: 50px" :src="item.fHead">{{item.fName}}
                            </p>
                        </router-link>
                    </div>
                    <div v-else-if="item.type == 'pic'">
                        <div class="left-name">{{item.userName}}</div>
                        <p class="text" v-more>
                            <img style="width: 200px" fit="widthFix" :src="item.fileUrl">
                        </p>
                    </div>
                    <div v-else-if="item.type == 'file'">
                        <div class="left-name">{{item.userName}}</div>
                        <p class="text" v-more @click.stop="downloadFile(item.fileName)">
                            <i style="font-size: 50px" class="el-icon-folder-remove"></i>
                        </p>
                    </div>
                    <div v-else>
                        <div class="left-name">{{item.userName}}</div>
                        <p class="text" v-more>{{item.value}}</p>
                    </div>
                </div>
                <div v-else>
                    <img :src="item.userHead" class="cheader">
                    <div v-if="item.type == 'friend'">
                        <router-link :to="{path:'/contact/details',query:{wxid:item.value}}" tag="div">
                            <p class="ctext" v-more>
                                <img style="width: 50px" :src="item.fHead">{{item.fName}}
                            </p>
                        </router-link>
                    </div>
                    <div v-else-if="item.type == 'pic'">
                        <p class="ctext" v-more>
                            <img style="width: 200px" fit="widthFix" :src="item.fileUrl">
                        </p>
                    </div>
                    <div v-else-if="item.type == 'file'">
                        <p class="ctext" v-more @click.stop="downloadFile(item.fileName)">
                            <i style="font-size: 50px" class="el-icon-folder-remove"></i>
                        </p>
                    </div>
                    <div v-else>
                        <p class="ctext" v-more>{{item.value}}</p>
                    </div>
                </div>
            </div>
        </section>
        <el-dialog class="dialog" width="80%" title="选择联系人" :visible.sync="dialogFormVisible" append-to-body>
            <el-form>
                <el-form-item>
                    <el-select v-model="friend" placeholder="请选择">
                        <el-option
                                v-for="item in $store.state.friends"
                                :key="item.userId"
                                :label="item.userName"
                                :value="item.userId">
                            <img width="30px" :src="item.userHead">
                            <span style="float: right; color: #8492a6; font-size: 13px">{{ item.userName }}</span>
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="success" @click="sendFriend">确 定</el-button>
            </div>
        </el-dialog>
        <footer class="dialogue-footer">
            <div class="component-dialogue-bar-person">
                <span class="iconfont icon-dialogue-voice"></span>
                <div class="chat-way">
                    <input class="chat-txt" v-model="text" type="text" />
                </div>
                <a href="javascript:;" class="weui-btn weui-btn_mini weui-btn_primary" @click="sendMessage()" style="width: 18%;margin: 5px 0 0 5px;">发送</a>
                <el-popover
                        placement="top"
                        width="200"
                        trigger="click">
                    <div class="icon" @click="dialogFormVisible = !dialogFormVisible">
                        <i class="el-icon-user-solid"></i>
                    </div>
                    <div class="icon" >
                        <el-upload style="display: inline;" class="avatar-uploader"
                                   action
                                   :http-request="handleUploadForm"
                                   :show-file-list="false"
                                   :before-upload="beforeAvatarUpload">
                            <i class="el-icon-picture"></i>
                        </el-upload>
                    </div>
                    <div class="icon" >
                        <el-upload style="display: inline;" class="avatar-uploader"
                                   action
                                   :http-request="fileHandleUploadForm"
                                   :show-file-list="false">
                            <i class="el-icon-folder-remove"></i>
                        </el-upload>
                    </div>
                    <span slot="reference" class="more iconfont icon-dialogue-jia"></span>
                </el-popover>
            </div>
        </footer>
    </div>
</template>
<script>
    import axios from "axios";

    export default {
        data() {
            return {
              text: '',
              info: [],
              isFriend: this.$route.query.isFriend,
              pageName: this.$route.query.name,
              dialogFormVisible: false,
              friend: ''
            }
        },
      created() {
          this.initPage()
      },
      mounted() {
        this.timer = setInterval(this.getMessage, 1000);
      },
      beforeDestroy() {
        clearInterval(this.timer);
      },
      methods: {
        goto() {
          this.$router.push({
            path: 'dialogue/dialogue-detail',
            query: {
              group_id: this.$route.query.friend
            }
          })
        },
        downloadFile(fileName) {
          window.location.href='http://localhost:8070/file/downloadFile/' + fileName;
        },
        fileHandleUploadForm(param) {
          var that = this
          let formData = new FormData();
          formData.append('userId', this.$store.state.user.userId)
          formData.append('file', param.file)
          axios.post('file/upload',formData).then(function(res){
            that.sendFile(res.data.fileId);
          })
        },
        handleUploadForm(param) {
          var that = this
          let formData = new FormData();
          formData.append('userId', this.$store.state.user.userId)
          formData.append('file', param.file)
          axios.post('file/upload',formData).then(function(res){
            that.sendPic(res.data.fileId);
          })
        },
        beforeAvatarUpload(file) {
          const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
          const isLt2M = file.size / 1024 / 1024 < 2;
          if (!isJPG) {
            this.$toast('发送图片只能是 JPG 格式!');
          }
          if (!isLt2M) {
            this.$toast('发送图片大小不能超过 2MB!');
          }
          return isJPG && isLt2M;
        },
        sendFile(fileId) {
          var that = this
          axios.post('/message/sendMessage', {
            userId: that.$store.state.user.userId,
            friend: that.$route.query.friend,
            value: fileId,
            type: 'file',
            isFriend: that.isFriend
          }).then(function(res) {
            if (res.data.success) {
              that.initPage()
            }
          })
        },
        sendPic(fileId) {
          var that = this
          axios.post('/message/sendMessage', {
            userId: that.$store.state.user.userId,
            friend: that.$route.query.friend,
            value: fileId,
            type: 'pic',
            isFriend: that.isFriend
          }).then(function(res) {
            if (res.data.success) {
              that.initPage()
            }
          })
        },
        sendFriend() {
          var that = this
          axios.post('/message/sendMessage',{
            userId: that.$store.state.user.userId,
            friend: that.$route.query.friend,
            value: that.friend,
            type: 'friend',
            isFriend: that.isFriend
          }).then(function (res) {
            if (res.data.success) {
              that.friend = ''
              that.initPage()
            }
          })
          this.dialogFormVisible = !this.dialogFormVisible
        },
          sendMessage() {
            var that = this
            axios.post('/message/sendMessage',{
              userId: that.$store.state.user.userId,
              friend: that.$route.query.friend,
              value: that.text,
              type: 'text',
              isFriend: that.isFriend
            }).then(function (res) {
              if (res.data.success) {
                that.text = ''
                that.initPage()
              }
            })
          },
          getMessage() {
            var that = this
            axios.post('/message/getMessage',{
              userId: that.$store.state.user.userId,
              friend: that.$route.query.friend,
              isFriend: that.isFriend
            }).then(function (res) {
              if (res.data.success) {
                that.info = res.data.message
              }
            })
          },
          initPage() {
            this.getMessage()
          }
      },
        beforeRouteEnter(to, from, next) {
            next(vm => {
                vm.$store.commit("setPageName", vm.$route.query.name)
            })
        },
        computed: {

        },
        directives: {
            press: {
                bind(element, binding) {
                    var startTx, startTy
                    element.addEventListener('touchstart', function(e) {
                        // 为什么每次注册监听器,都要重新获取一次 DOM 像上面写就 undefine?
                        var recording = document.querySelector('.recording'),
                            recordingVoice = document.querySelector('.recording-voice')
                        element.className = "chat-say say-active"
                        recording.style.display = recordingVoice.style.display = "block"
                        var touches = e.touches[0]
                        startTx = touches.clientX
                        startTy = touches.clientY
                        e.preventDefault()
                    }, false)
                    element.addEventListener('touchend', function(e) {
                        var recording = document.querySelector('.recording'),
                            recordingVoice = document.querySelector('.recording-voice'),
                            recordingCancel = document.querySelector('.recording-cancel')
                        element.className = "chat-say"
                        recordingCancel.style.display = recording.style.display = recordingVoice.style.display = "none"
                        e.preventDefault()
                    }, false)
                    element.addEventListener('touchmove', function(e) {
                        var recording = document.querySelector('.recording'),
                            recordingVoice = document.querySelector('.recording-voice'),
                            recordingCancel = document.querySelector('.recording-cancel')
                        var touches = e.changedTouches[0],
                            endTx = touches.clientX,
                            endTy = touches.clientY,
                            distanceX = startTx - endTx,
                            distanceY = startTy - endTy;
                        if (distanceY > 10 && distanceY < 80) {
                            // 控制范围 和谐掉指尖抖动
                            element.className = "chat-say"
                            recordingVoice.style.display = "none"
                            recordingCancel.style.display = "block"
                        } else if (distanceY > 80) {
                            element.className = "chat-say"
                            recording.style.display = recordingCancel.style.display = "none"
                        }
                        // 阻断事件冒泡 防止页面被一同向上滑动
                        e.preventDefault()
                    }, false);
                }
            },
            more: {
                bind(element, binding) {
                    var startTx, startTy
                    element.addEventListener('touchstart', function(e) {
                        var msgMore = document.getElementById('msg-more'),
                            touches = e.changedTouches[0],
                            startTx = touches.clientX,
                            startTy = touches.clientY
                            // 控制菜单的位置
                        msgMore.style.left = ((startTx - 18) > 180 ? 180 : (startTx - 18)) + 'px'
                        msgMore.style.top = (element.offsetTop - 33) + 'px'
                        msgMore.style.display = "block"
                        e.preventDefault()
                    }, false)
                    element.addEventListener('touchend', function(e) {
                        e.preventDefault()
                    }, false)
                }
            }
        }
    }
</script>
<style>
    @import "../../assets/css/dialogue.css";
    .say-active {
        background: #c6c7ca;
    }
    .icon {
        font-size: 40px;
        display: inline;
        width: 50px;
        height: 50px;
    }
    .cheader{
        width:35px;
        float:right;
        display:block;
        position:relative;
        left: 60px;
    }
    .ctext{
        float:right;
        background:#9eea6a;
        padding:8px;
        box-sizing:border-box;
        margin-left:10px;
        position:relative;
        border-radius:4px;
        max-width:80%;
        font-size:14px;
        left: 50px;
    }
    .dialogue-section .row .ctext:before {
        width: 0;
        height: 0;
        position: absolute;
        right: -12px;
        top: 11px;
        content: "";
        border: 6px solid transparent;
        border-left-color: #9eea6a;
    }
    .left-name {
        color: gray;
    }
</style>
