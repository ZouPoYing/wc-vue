<template>
    <div>
        <header id="wx-header">
            <!--<div class="other"><span>添加朋友</span></div>-->
            <div class="center">
                <div class="iconfont icon-return-arrow" v-on:click="$router.back()">
                    <span>返回</span>
                </div>
                <span>聊天详情</span>
            </div>
        </header>
        <div class="member">
            <ul class="chat-dialogue-entry-collect">
                <li v-for="(item,index) in group.friendDetails" :key="index">
                    <div><img :src="item.friendHead"></div>
                    <p>{{item.friendName}}</p>
                </li>
                <li>
                    <div><span class="iconfont icon-chat-detail-add"></span></div>
                </li>
            </ul>
        </div>
        <div class="weui-cells">
            <div class="weui-cell weui-cell_access" @click="pop">
                <div class="weui-cell__bd">群名称</div>
                <div class="weui-cell__ft">{{group.groupName}}</div>
            </div>
        </div>
        <el-dialog class="dialog" width="80%" title="修改群名称" :visible.sync="dialogFormVisible" append-to-body>
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
  import axios from "axios";

    export default {
      data() {
        return {
          group: [],
          dialogFormVisible: false,
          form: {
            msg: ''
          }
        }
      },
      created () {
        this.initPage()
      },
      methods: {
        change() {
          var that = this
          axios.post('/group/updateGroupName', {
            groupId: that.$route.query.group_id,
            groupName: that.form.msg,
            userId: that.$store.state.user.userId
          }).then(function(res) {
            if (res.data.success) {
              that.getGroupDetail()
            } else {
              that.$toast(res.data.msg);
            }
          })
          that.pop()
        },
        getGroupDetail() {
          var that = this
          axios.post('/group/getGroupDetail', {
            groupId: that.$route.query.group_id
          }).then(function(res) {
            if (res.data.success) {
              that.group = res.data.group
            }
          })
        },
        pop() {
          this.dialogFormVisible = !this.dialogFormVisible
        },
        initPage() {
          this.getGroupDetail()
        }
      },
      mounted(){
        }
    }

</script>
<style>
    .chat-dialogue-entry-collect {
        background-color: #fff;
        position: relative;
        padding: 15px 10px 5px;
        display: flex;
        flex-wrap: wrap;
        justify-content: flex-start;
        align-items: flex-start;
        width: 100%;
        color: #464646;
        font-size: 14px;
    }

    .chat-dialogue-entry-collect:before {
        content: "";
        position: absolute;
        width: 200%;
        left: 0;
        bottom: 0;
        transform: scale(.5);
        transform-origin: 0 0;
        -webkit-transform: scale(.5);
        -webkit-transform-origin: 0 0;
        background-color: #b7b7b7;
        height: 1px;
        z-index: 2;
    }

    .chat-dialogue-entry-collect li {
        float: left;
        flex-grow: 1;
        flex-basis: 25%;
        max-width: 25%;
        padding: 5px 10px;
        text-align: center;
    }

    .chat-dialogue-entry-collect li>div {
        position: relative;
        border-radius: 6px;
        overflow: hidden;
        width: 55px;
        height: 55px;
        margin: 0 auto;
        background-size: cover;
        border: 1px solid #eee;
    }

    .chat-dialogue-entry-collect li>div img {
        width: 100%;
    }

    .chat-dialogue-entry-collect li p {
        margin-top: 5px;
    }

    .chat-dialogue-entry-collect li .iconfont {
        font-size: 23px;
        color: #bbb;
        line-height: 55px
    }
</style>
