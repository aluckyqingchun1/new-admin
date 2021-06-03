<template>
  <box :sub-title="subTitle">
    <el-form ref="form" class="login-form" :model="form" :rules="rules">
      <el-form-item prop="contactPhone">
        <i class="icon phone" />
        <el-input
          ref="contactPhone"
          v-model="form.contactPhone"
          placeholder="请输入手机号"
          autocomplete="on"
          tabindex="1"
          maxlength="11"
        />
      </el-form-item>
      <template v-if="!isForget">
        <el-form-item prop="password">
          <password :value.sync="form.password" />
        </el-form-item>
        <el-form-item prop="contactName">
          <i class="icon user" />
          <el-input v-model="form.contactName" tabindex="3" placeholder="请输入联系人姓名" />
        </el-form-item>
      </template>
      <template v-else>
        <el-form-item prop="password">
          <i class="icon password" />
          <el-input
            v-model="form.password"
            type="password"
            tabindex="2"
            maxlength="16"
            placeholder="请输入新密码"
          />
        </el-form-item>
        <el-form-item prop="confirmPwd">
          <i class="icon re-password" />
          <el-input
            v-model="form.confirmPwd"
            type="password"
            tabindex="3"
            maxlength="16"
            placeholder="请再次输入新密码"
          />
        </el-form-item>
      </template>
      <el-form-item prop="imageCode">
        <i class="icon img_code" />
        <verification-code ref="vCode" :form.sync="form" />
      </el-form-item>
      <el-form-item prop="smsCode">
        <i class="icon msg" />
        <message-code :form.sync="form" :mobile="form.contactPhone" @checkSend="checkSend" />
      </el-form-item>
      <el-button
        :loading="loading"
        size="default"
        type="primary"
        class="lrf-btn"
        @click.native.prevent="handleClick"
      >{{ isForget ? '提 交': '注 册' }}</el-button>
    </el-form>
  </box>
</template>

<script>
import Box from './components/Box'
import Password from '@/components/Password'
import VerificationCode from '@/components/VerificationCode'
import MessageCode from '@/components/MessageCode'
import { isTel, password } from '@/utils/validate'
import { register, resetPwd, getChannelDetail } from '@/api/user'
import { mapActions } from 'vuex'

export default {
  name: 'Register',
  components: { Box, Password, VerificationCode, MessageCode },
  data() {
    const validatePhone = (rule, value, callback) => {
      if (!isTel(value)) {
        callback(new Error('手机号格式错误'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (!value) {
        callback(new Error(`${this.isForget ? '请输入新密码' : '请输入密码'}`))
      } else if (!password(value)) {
        callback(new Error('密码为8-16位的字母、数字或符号组合'))
      } else {
        if (this.isForget && this.form.confirmPwd !== '') {
          this.$refs.form.validateField('confirmPwd')
        }
        callback()
      }
    }
    const validatePassword2 = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请再次输入新密码'))
      } else if (this.form.password !== this.form.confirmPwd) {
        callback(new Error('两次输入密码不一致'))
      } else {
        callback()
      }
    }
    return {
      form: {
        contactPhone: '', // 手机号
        password: '', // 密码
        confirmPwd: '', // 确认密码
        contactName: '', // 联系人
        imageCode: '', // 图形验证码
        imageKey: '', // 验证码key
        smsCode: '', // 短信验证码
        smsKey: '' // 短信验证码key
      },
      rules: {
        contactPhone: [
          {
            required: true,
            whitespace: true,
            message: '请输入手机号',
            trigger: 'blur'
          },
          {
            validator: validatePhone,
            trigger: 'blur'
          }
        ],
        password: [{ validator: validatePassword, trigger: 'blur' }],
        confirmPwd: [{ validator: validatePassword2, trigger: 'blur' }],
        contactName: [
          {
            required: true,
            whitespace: true,
            message: '请输入联系人姓名',
            trigger: 'blur'
          }
        ],
        imageCode: [
          { required: true, message: '请输入图片验证码', trigger: 'blur' }
        ],
        smsCode: [
          { required: true, message: '请输入短信验证码', trigger: 'blur' }
        ]
      },
      loading: false
    }
  },
  computed: {
    // ...mapGetters('channel', ['no']),
    isForget: vm => vm.$route.name === 'forget',
    subTitle: vm => (vm.isForget ? '修改密码' : '注册')
  },
  created() {
    const cn = this.$route.query.channelNo
    if (cn) {
      this.setNo(cn)
      getChannelDetail({ channelNo: cn }).then(({ data }) => {
        if (data && data.logo) {
          this.setLogo(data.logo)
        }
      })
    }
  },
  mounted() {
    if (this.$refs.contactPhone) {
      this.$refs.contactPhone.focus()
    }
  },
  methods: {
    ...mapActions('channel', ['setNo', 'setLogo']),
    ...mapActions('user', ['setLogin']),
    // 发送验证码的回调执行函数
    checkSend(fn) {
      let mFlag = false
      let iFlag = false
      this.$refs.form.validateField('contactPhone', errorMessage => {
        if (!errorMessage) {
          mFlag = true
        } else {
          mFlag = false
        }
      })
      this.$refs.form.validateField('imageCode', errorMessage => {
        if (!errorMessage) {
          iFlag = true
        } else {
          iFlag = false
        }
      })

      setTimeout(() => {
        if (mFlag && iFlag) {
          fn && fn()
        }
      }, 0)
    },
    handleClick() {
      this.$refs.form.validate(valid => {
        if (valid) {
          if (
            this.form.cachePhone &&
            this.form.cachePhone !== this.form.contactPhone
          ) {
            this.$message.error('手机号与获取短信验证码的手机号不一致')
            return false
          }
          this.loading = true
          if (!this.isForget) {
            register(
              Object.assign({}, this.form, {
                // channelNo: this.no ? this.no : undefined,
                confirmPwd: undefined
              })
            )
              .then(({ data }) => {
                this.successCb('注册成功！', () => {
                  this.setLogin({
                    ticket: data.ticket,
                    userId: data.userId,
                    name: this.form.contactPhone
                  })
                  this.$router.push({ name: 'homepage' })
                })
              })
              .catch(() => {
                this.loading = false
              })
          } else {
            // 去除无用参数
            resetPwd(
              Object.assign({}, this.form, {
                contactName: undefined,
                confirmPwd: undefined
              })
            )
              .then(({ data }) => {
                this.successCb('修改密码成功！', () => {
                  this.$router.push({ name: 'login' })
                })
              })
              .catch(() => {
                this.loading = false
              })
          }
        }
      })
    },
    successCb(message, cb) {
      const msg = this.$message.success(message)
      setTimeout(() => {
        msg.close()
        this.loading = false
        cb && cb()
      }, 1500)
    }
  }
}
</script>

<style lang='scss' scoped>
</style>
