<template>
    <div id="app">
        <div v-if="!this.$store.state.isLogin" class="loginBack">
            <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="60px" class="demo-ruleForm">
                <el-form-item v-if="!this.$store.state.isLoginPage" label="昵称" prop="name">
                    <el-input v-model="ruleForm.name"></el-input>
                </el-form-item>
                <el-form-item label="手机号" prop="phone">
                    <el-input v-model="ruleForm.phone"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input type="password"  v-model="ruleForm.password"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button v-if="this.$store.state.isLoginPage" class="btn-zc" type="success" @click="login">登录</el-button>
                    <el-button v-else class="btn-zc" type="success" @click="register">注册</el-button>
                </el-form-item>
            </el-form>
            <el-tag v-if="this.$store.state.isLoginPage" @click="qzc" class="qdl">去注册</el-tag>
            <el-tag v-else @click="qzc" class="qdl">去登录</el-tag>
        </div>
        <div v-else>
            <div  class="outter" :class="{'hideLeft':$route.path.split('/').length>2}">
                <!--通用头部-->
                <header class="app-header" :class="{'header-hide':!$store.state.headerStatus}">
                    <wx-header :pageName="pageName"></wx-header>
                </header>
                <!--搜索框 只在“微信”和“通讯录”页面下显示-->
                <search v-show="$route.path.indexOf('explore')===-1&&$route.path.indexOf('self')===-1"></search>
                <!--四个门面页 “微信” “通讯录” “发现” “我”-->
                <section class="app-content">
                    <keep-alive>
                        <router-view name="default"></router-view>
                    </keep-alive>
                </section>
                <!--底部导航 路由 -->
                <footer class="app-footer">
                    <wx-nav></wx-nav>
                </footer>
            </div>
            <!--其他店内页集合 有过渡效果-->
            <transition name="custom-classes-transition" :enter-active-class="enterAnimate" :leave-active-class="leaveAnimate">
                <router-view name="subPage" class="sub-page"></router-view>
            </transition>
        </div>
    </div>
</template>

<script>
    import axios from "axios"
    import WxHeader from './components/common/wx-header'
    import WxNav from './components/common/wx-nav'
    import search from './components/common/search'
    import mixin from "./vuex/mixin.js" // 混合被单独放在 mixin.js 中管理
    window.mixin = mixin // 将 混合/mixin 暴露在窗口对象中，某些组件需要时，直接提取 window.mixin 
    export default {
        name: 'app',
        components: {
            WxHeader,
            WxNav,
            search
        },
        data() {
            return {
                ruleForm: {
                    name: '',
                    phone: '',
                    password: ''
                },
                "pageName": "",
                "routerAnimate": false,
                "enterAnimate": "", //页面进入动效
                "leaveAnimate": "" //页面离开动效
            }
        },
      methods: {
          register() {
              var that = this
              axios.post('/user/register', {
                userName: this.ruleForm.name,
                userPhone: this.ruleForm.phone,
                password: this.ruleForm.password
              }).then(function(res){
                if (res.data.success) {
                  that.$toast('注册成功');
                } else {
                  that.$toast(res.data.msg);
                }
              })
           },
          login() {
            var that = this
            axios.post('/user/login', {
              userPhone: this.ruleForm.phone,
              password: this.ruleForm.password
            }).then(function(res){
              if (res.data.success) {
                that.$toast('登录成功');
                that.$store.commit('setUser',res.data.user);
                that.$store.commit('setIsLogin',true);
                that.getFriend()
              } else {
                that.$toast(res.data.msg);
              }
            })
          },
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
          qzc() {
            this.$store.commit('setIsLoginPage',!this.$store.state.isLoginPage);
          }
        },
        watch: {
            // 监听 $route 为店内页设置不同的过渡效果
            "$route" (to, from) {
                const toDepth = to.path.split('/').length
                const fromDepth = from.path.split('/').length
                if (toDepth === 2) {
                    this.$store.commit("setPageName", to.name)
                }
                //同一级页面无需设置过渡效果
                if (toDepth === fromDepth) {
                    return;
                }
                this.enterAnimate = toDepth > fromDepth ? "animated fadeInRight" : "animated fadeInLeft"
                this.leaveAnimate = toDepth > fromDepth ? "animated fadeOutLeft" : "animated fadeOutRight"
                    // 从店面页进入店内页 需要对店内页重新设置离开动效 因为他们处于不同 name 的 router-view
                if (toDepth === 3) {
                    this.leaveAnimate = "animated fadeOutRight"
                }
            }
        }
    }
</script>
<style>
    /*将公用的样式统一在此导入*/
    
    @import "assets/css/base.css";
    @import "assets/css/common.css";
    @import "assets/css/wx-header.css";
    /*阿里 fonticon*/
    
    @import "assets/css/lib/iconfont.css";
    /*过渡效果需要的动画库*/
    
    @import "assets/css/lib/animate.css";
    /*weui 样式库 非常适合高仿微信*/
    
    @import "assets/css/lib/weui.min.css";
    .demo-ruleForm {
        margin: 150px 20px 150px 20px;
    }

    .btn-zc {
        width: 250px;
    }

    .qdl {
        margin-left: 250px;
    }
</style>