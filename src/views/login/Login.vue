<template>
  <box class="box-login">
    <el-form ref="loginForm" class="login-form" :model="loginForm" :rules="loginRules">
      <el-form-item prop="loginAcct">
        <i class="icon phone" />
        <el-input
          ref="username"
          v-model="loginForm.loginAcct"
          placeholder="用户名"
          tabindex="1"
          autocomplete="on"
        />
      </el-form-item>
      <el-tooltip v-model="capsTooltip" content="大写锁定已打开" placement="top" manual>
        <el-form-item prop="password">
          <i class="icon password" />
          <el-input
            :key="passwordType"
            ref="password"
            v-model="loginForm.password"
            :type="passwordType"
            placeholder="密码"
            tabindex="2"
            autocomplete="on"
            @keyup.native="checkCapslock"
            @blur="capsTooltip = false"
          />
          <span class="show-pwd" @click="showPwd">
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
          </span>
        </el-form-item>
      </el-tooltip>
      <el-form-item prop="captcha">
        <template>
          <captcha :form="loginForm" />
        </template>
      </el-form-item>
      <el-form-item>
        <el-button
          :loading="loading"
          size="default"
          type="primary"
          class="lrf-btn"
          @click.native.prevent="handleLogin"
        >登 录</el-button>
      </el-form-item>
    </el-form>
  </box>
</template>

<script>
// import VerificationCode from '@/components/verifition/Verify.vue'
import Box from './components/Box'
import captcha from './components/captcha'

import { password } from '@/utils/validate'

export default {
  name: 'Login',
  components: {
    Box,
    captcha
  },
  data() {
    const validatePassword = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请输入密码'))
      } else if (!password(value)) {
        callback(new Error('密码为8-16位的字母、数字或符号组合'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        loginAcct: '',
        password: ''
      },
      loginRules: {
        loginAcct: [
          { required: true, trigger: 'blur', message: '请输入用户名' }
        ],
        password: [
          { required: true, trigger: 'blur', validator: validatePassword }
        ]
        // imageCode: [
        //   { required: true, trigger: 'blur', message: '请输入图片验证码' }
        // ]
      },
      passwordType: 'password',
      capsTooltip: false,
      loading: false,
      redirect: undefined,
      otherQuery: {},
      verify: false,
      verifyed: false,
      captchaVerification: ''
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  mounted() {
    if (this.loginForm.username === '') {
      this.$refs.username.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  methods: {
    // 遮挡层  让滑块验证码消失
    cancelVerify() {
      this.verify = false
    },
    showVerification() {
      this.verify = true
    },
    success(params) {
      // params 返回的二次验证参数
      if (params) {
        this.verify = false
        this.verifyed = true
        this.captchaVerification = params
      }
    },
    checkCapslock({ shiftKey, key } = {}) {
      if (key && key.length === 1) {
        if (
          (shiftKey && key >= 'a' && key <= 'z') ||
          (!shiftKey && key >= 'A' && key <= 'Z')
        ) {
          this.capsTooltip = true
        } else {
          this.capsTooltip = false
        }
      }
      if (key === 'CapsLock' && this.capsTooltip === true) {
        this.capsTooltip = false
      }
    },
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          if (!this.captchaVerification) {
            this.$message({
              message: '请先完成安全验证',
              type: 'warning'
            })
            return
          }
          this.loading = true
          this.$store
            .dispatch(
              'user/login',
              Object.assign({}, this.loginForm, this.captchaVerification, {
                userType: 'ODP'
              })
            )
            .then((res) => {
              if ((res && res.status === '1005') || (res && res.data.status === '2006')) {
                this.$message.error('安全码校验失效，请重新校验')
                this.loading = false
                this.verify = false
                this.verifyed = false
                this.captchaVerification = ''
                return
              }
              this.$router.push({
                path: this.redirect || '/',
                query: this.otherQuery
              })
              this.loading = false
            })
            .catch(err => {
              this.verify = false
              this.verifyed = false
              this.captchaVerification = ''
              console.log(err)
              this.loading = false
            })
        } else {
          this.$message.error('请重新提交')
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    },
    forget() {
      this.$router.push({ name: 'forget' })
    }
  }
}
</script>

<style lang="scss" scoped>
.icon{
    border: none !important;
}
.show-pwd {
  position: absolute;
  right: 10px;
  top: 10px;
  font-size: 16px;
  color: #889aa4;
  cursor: pointer;
  user-select: none;
}
span.forget {
    display: inline-block;
    font-size: 12px;
    font-family: Microsoft YaHei;
    margin-left: 240px;
    margin-bottom: 10px;
    font-weight: 400;
    line-height: 49px;
    color: #387EE8;
    cursor: pointer;
}
.img-code{
    text-align: center;
    width: 301px;
    height: 40px;
    font-size: 14px;
    line-height: 40px;
    background: #F7F9FA;
    border: 1px solid #E4E7EB;
    opacity: 1;
    font-family: Microsoft YaHei;
    font-weight: 400;
    color: rgba(0, 0, 0, 0.85);
}
.img-code:hover{
    color: #36D1DC;
    cursor: pointer;
}
.pop-specil{
    display: none;
}
.complete{
    text-align: center;
    width: 301px;
    height: 40px;
    font-size: 14px;
    line-height: 40px;
    background:#52ccba;
    opacity: 0.1;
    border: 1px solid #E4E7EB;
    opacity: 1;
    font-family: Microsoft YaHei;
    font-weight: 400;
    color: azure;
    pointer-events: none;
}
.show-verify{
    position: relative;
    .verify{
        position: absolute;
        bottom: 0px;
        z-index: 999;
    }
}
.mask{
z-index: 888;
}
</style>
<style lang="scss">
// 小屏
.box-login{
   .login-form .el-input input{
        height: 40px !important;
    }
}
</style>
