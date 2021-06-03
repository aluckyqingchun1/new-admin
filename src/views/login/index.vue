<template>
  <section class="container">
    <div class="box" :class="{ 'forget' : isForget }">
      <img class="aside" src="@/assets/images/login/logo.png" alt="侧边图">
      <component :is="currentRole" class="box-wrapper" />
    </div>
    <img class="footer-img" src="@/assets/images/login/footer.png" alt>
    <!-- <div class="footer-bg" /> -->
    <footer>Copyright&copy;{{ `${new Date().getFullYear()} 深圳前海九慧金服科技有限公司` }}</footer>
  </section>
</template>

<script>
import Login from './Login'
// import Register from './Register'

export default {
  name: 'IndexLayout',
  components: { Login },
  data() {
    return {
      tempLogo: require('@/assets/images/logo.png')
    }
  },

  computed: {
    currentRole: vm => {
      console.log(vm, '8888888888888')
      const n = vm.$route.name
      if (n === 'login') {
        return 'Login'
      } else if (n === 'register' || n === 'forget') {
        return 'Register'
      } else {
        return ''
      }
    },
    isForget: vm => vm.$route.name === 'forget'
  },
  methods: {
    handelClick() {
      if (this.currentRole === 'Login') {
        this.$router.push({ name: 'register' })
      } else if (this.currentRole === 'Register') {
        this.$router.push({ name: 'login' })
      }
    }
  }
}
</script>

<style lang="scss">
$bg: #000;

/* reset element-ui css */
.login-form {
  .el-input {
    input {
      font-size: 14px;
      border: 0;
      -webkit-appearance: none;
      color: $bg;
      height: 45px;
      caret-color: $bg;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px #fff inset !important;
        -webkit-text-fill-color: $bg !important;
      }
    }
  }

  .lrf-btn {
    width: 100%;
    margin-top: 35px;
    font-size: 14px;
    background-color: #387EE8;
  }

  .el-form-item {
    border-bottom: 1px solid #ccc;
    color: #454545;

    .el-form-item__content {
      display: flex;
      align-items: center;

      i.icon {
        display: block;
        width: 26px;
        height: 24px;
        background-repeat: no-repeat;
        flex-grow: 1;

        &.phone {
          background-image: url("~@/assets/images/phone_ico.png");
        }
        &.password {
          background-image: url("~@/assets/images/pwd_ico.png");
        }
        &.re-password {
          background-image: url("~@/assets/images/pwd_ico2.png");
        }
        &.img_code {
          background-image: url("~@/assets/images/code_ico.png");
        }
        &.user {
          background-image: url("~@/assets/images/user_ico.png");
        }
        &.msg {
          background-image: url("~@/assets/images/msg_ico.png");
        }
      }
    }

    .el-form-item__error {
      padding-top: 2px;
    }
  }
}
</style>

<style lang="scss" scoped>
.container {
  min-height: 100%;
  // background-color: #fff;
  background-image: url("~@/assets/images/login/bg.png");

  header {
    height: 80px;
    box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.1);

    & > div {
      display: flex;
      max-width: 900px;
      height: 100%;
      margin: 0 auto;
      justify-content: space-between;
      align-items: center;
    }
  }

  footer {
    position: fixed;
    bottom: 0;
    width: 100%;
    height: 50px;
    line-height: 50px;
    text-align: center;
    color: #808080;
    font-size: 12px;
    background-color: transparent;
  }
}

.box {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 100px;
  margin: 0 auto;
  width: 1000px;
  // margin: 130px auto 0;

  img.aside {
    display: block;
    max-width: 450px;
    // height: 786px;
    // margin-top: 48px;
  }

  &.forget {
    justify-content: center;

    img.aside {
      display: none;
    }
  }
}

.box-wrapper {
  position: relative;
  z-index: 2;
  background-color: #fff;
}

.footer-img{
   position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
}

// 小屏
@media (max-width: 700px) {
    .container{
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    .box{
  padding-top: 0px;
  width: 380px;
    }
    img.aside {
      display: none;
    }

    // div.login-form-box {
    //   margin: 0 auto;
    // }
  }
}
</style>
