<template>
  <div class="main">
    <ul class="bg-bubbles">
      <li v-for="i in 15" :key="i"></li>
    </ul>
    <div class="login-container">
      <div class="login-wrapper">
        <div class="login-img">
          <img src="~@/assets/images/img-01.png" alt="" />
        </div>
        <el-form
          ref="loginForm"
          :model="loginForm"
          :rules="loginRules"
          class="login-form"
        >
          <h2 class="title">在线考试系统</h2>
          <el-form-item prop="username">
            <el-input
              v-model="loginForm.username"
              type="text"
              auto-complete="off"
              placeholder="账号"
            >
              <svg-icon
                slot="prefix"
                icon-class="user"
                class="el-input__icon input-icon"
              />
            </el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              v-model="loginForm.password"
              type="password"
              auto-complete="off"
              placeholder="密码"
              @keyup.enter.native="handleLogin"
            >
              <svg-icon
                slot="prefix"
                icon-class="password"
                class="el-input__icon input-icon"
              />
            </el-input>
          </el-form-item>
          <!-- <el-form-item prop="code">
            <el-input
              v-model="loginForm.code"
              auto-complete="off"
              placeholder="验证码"
              style="width: 63%"
              @keyup.enter.native="handleLogin"
            >
              <svg-icon
                slot="prefix"
                icon-class="validCode"
                class="el-input__icon input-icon"
              />
            </el-input>
            <div class="login-code">
              <img :src="codeUrl" @click="getCode" class="login-code-img" />
            </div>
          </el-form-item> -->
          <!-- <el-checkbox
            v-model="loginForm.rememberMe"
            style="margin:0px 0px 25px 0px;"
            >记住密码</el-checkbox
          > -->
          <el-form-item style="width:100%;">
            <el-button
              :loading="loading"
              size="medium"
              type="primary"
              style="width:100%;"
              @click.native.prevent="handleLogin"
            >
              <span v-if="!loading">登 录</span>
              <span v-else>登 录 中...</span>
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import { getCodeImg } from "@/api/login";
import Cookies from "js-cookie";
// import { encrypt, decrypt } from "@/utils/jsencrypt";

export default {
  name: "Login",
  data() {
    return {
      codeUrl: "",
      cookiePassword: "",
      loginForm: {
        username: "admin",
        password: "",
        rememberMe: false,
        code: "",
        uuid: ""
      },
      loginRules: {
        username: [
          { required: true, trigger: "blur", message: "用户名不能为空" }
        ],
        password: [{ required: true, trigger: "blur", message: "密码不能为空" }]
        // code: [{ required: true, trigger: "change", message: "验证码不能为空" }]
      },
      loading: false,
      redirect: undefined
    };
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect;
      },
      immediate: true
    }
  },
  created() {
    // TODO 验证码暂时注释
    // this.getCode();
    // this.getCookie();
  },
  methods: {
    getCode() {
      getCodeImg().then(res => {
        this.codeUrl = "data:image/gif;base64," + res.img;
        this.loginForm.uuid = res.uuid;
      });
    },
    getCookie() {
      const username = Cookies.get("username");
      const password = Cookies.get("password");
      const rememberMe = Cookies.get("rememberMe");
      this.loginForm = {
        username: username === undefined ? this.loginForm.username : username,
        password:
          password === undefined ? this.loginForm.password : decrypt(password),
        rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
      };
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true;
          // if (this.loginForm.rememberMe) {
          //   Cookies.set("username", this.loginForm.username, { expires: 30 });
          //   Cookies.set("password", encrypt(this.loginForm.password), {
          //     expires: 30
          //   });
          //   Cookies.set("rememberMe", this.loginForm.rememberMe, {
          //     expires: 30
          //   });
          // } else {
          //   Cookies.remove("username");
          //   Cookies.remove("password");
          //   Cookies.remove("rememberMe");
          // }
          this.$store
            .dispatch("Login", this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || "/" }).catch(() => {});
              setTimeout(() => {
                this.$notify({
                  title: "欢迎",
                  message: "您好，欢迎回来",
                  type: "success",
                  duration: 3000
                });
              }, 800);
            })
            .catch(() => {
              this.loading = false;
              // this.getCode();
            });
        }
      });
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss">
.main {
  width: 100%;
  margin: 0 auto;
}
.login-container {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  padding: 15px;
  background: linear-gradient(-135deg, #c850c0, #4158d0);
  z-index: 10;
}
.login-wrapper {
  width: 960px;
  height: 645px;
  background: #fff;
  border-radius: 10px;
  overflow: hidden;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  padding: 177px 130px 33px 95px;
}
.login-img {
  width: 316px;
}
.ant-form {
  width: 290px;
  // margin-top: 80px;
}
h2 {
  text-align: center;
  margin-bottom: 40px;
  font-size: 30px;
  font-weight: normal;
}
.bg-bubbles {
  position: absolute;
  // 使气泡背景充满整个屏幕；
  top: 0;
  left: 0;
  width: 100%;
  height: 98%;
  // 如果元素内容超出给定的宽度和高度，overflow 属性可以确定是否显示滚动条等行为；
  overflow: hidden;
  li {
    position: absolute;
    // bottom 的设置是为了营造出气泡从页面底部冒出的效果；
    bottom: -160px;
    // 默认的气泡大小；
    width: 40px;
    height: 40px;
    background-color: rgba(255, 255, 255, 0.15);
    list-style: none;
    // 使用自定义动画使气泡渐现、上升和翻滚；
    animation: square 15s infinite;
    transition-timing-function: linear;
    // 分别设置每个气泡不同的位置、大小、透明度和速度，以显得有层次感；
    // border-radius: 50%;
    &:hover {
      animation-play-state: paused;
      // transition: 0.1s all linear;
    }
    &:nth-child(1) {
      left: 10%;
      width: 120px;
      height: 120px;
      animation-delay: 6s;
      animation-duration: 7s;
    }
    &:nth-child(2) {
      left: 80%;
      width: 90px;
      height: 90px;
      animation-delay: 7s;
      animation-duration: 7s;
    }
    &:nth-child(3) {
      left: 17%;
      width: 160px;
      height: 160px;
      animation-delay: 4s;
    }
    &:nth-child(4) {
      left: 44%;
      width: 60px;
      height: 60px;
      animation-duration: 8s;
      background-color: rgba(255, 255, 255, 0.3);
    }
    &:nth-child(5) {
      left: 70%;
    }
    &:nth-child(6) {
      left: 80%;
      width: 120px;
      height: 120px;
      animation-delay: 3s;
      background-color: rgba(255, 255, 255, 0.2);
    }
    &:nth-child(7) {
      left: 90%;
      width: 160px;
      height: 160px;
      animation-delay: 2s;
    }
    &:nth-child(8) {
      left: 55%;
      width: 60px;
      height: 60px;
      animation-delay: 4s;
      animation-duration: 15s;
    }
    &:nth-child(9) {
      left: 70%;
      width: 80px;
      height: 80px;
      animation-delay: 2s;
      animation-duration: 12s;
      background-color: rgba(255, 255, 255, 0.3);
    }
    &:nth-child(10) {
      left: 65%;
      width: 160px;
      height: 160px;
      animation-delay: 5s;
    }
    &:nth-child(11) {
      left: 55%;
      width: 160px;
      height: 160px;
      animation-delay: 5s;
    }
    &:nth-child(12) {
      left: 5%;
      width: 160px;
      height: 160px;
      animation-delay: 4s;
    }
    &:nth-child(13) {
      left: 24%;
      width: 90px;
      height: 90px;
      animation-delay: 3s;
    }
    &:nth-child(14) {
      left: 89%;
      width: 90px;
      height: 90px;
      animation-delay: 2s;
    }
    &:nth-child(15) {
      left: 3%;
      width: 90px;
      height: 90px;
      animation-delay: 1s;
    }
  }
  // 自定义 square 动画；
  @keyframes square {
    0% {
      opacity: 0.5;
      transform: translateY(0px) rotate(45deg);
    }
    25% {
      opacity: 0.75;
      transform: translateY(-400px) rotate(90deg);
    }
    50% {
      opacity: 1;
      transform: translateY(-600px) rotate(135deg);
    }
    100% {
      opacity: 0;
      transform: translateY(-1000px) rotate(180deg);
    }
  }
}
</style>
