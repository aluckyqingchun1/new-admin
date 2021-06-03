<template>
  <div class="code-wrapper">
    <div class="input-container">
      <span class="svg-container">
        <!-- <svg-icon icon-class="user" /> -->
        <i class="el-icon-picture" />
      </span>
      <el-input
        v-model="form.imageCode"
        class="code-input"
        maxlength="4"
        placeholder="图片验证码"
        autocomplete="on"
      />
    </div>

    <div
      v-loading="loading"
      class="code-img"
      element-loading-spinner="el-icon-loading"
      @click="getImgCode"
      v-html="codeImg"
    >
      <img v-if="codeImg" alt="获取验证码">
      <div v-else class="code-empty" @click="getImgCode">获取验证码</div>
    </div>
  </div>
</template>

<script>
import { check } from '@/api/user'

export default {
  name: 'Picturecode',
  props: {
    form: {
      required: true,
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      codeImg: '',
      loading: false
    }
  },
  created() {
    this.getImgCode()
  },
  methods: {
    async   getImgCode() {
      this.loading = true
      const { data: img } = await check()
      this.codeImg = img
      this.loading = false
    }
  }
}
</script>

<style scoped lang="scss">
.code-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;

  .svg-container {
    width: 30px;
    font-size: 16px;
    padding: 6px 5px 6px 15px;
    color: #889aa4;

    i {
      vertical-align: middle;
    }
  }

  .input-container {
    display: flex;
    width: 65%;
    /*border: 1px solid rgba(255, 255, 255, 0.1);*/
    /*background: rgba(0, 0, 0, 0.1);*/
    /*border-radius: 5px;*/
    color: #454545;

    .code-input {
      flex-grow: 1;
    }
  }

  .code-img {
    width: 30%;
    cursor: pointer;
    height: 100%;

    img {
      display: block;
      width: 100%;
      height: 100%;
    }

    .code-empty {
      height: 100%;
      line-height: 40px;
      text-align: center;
      color: #eee;
      border: 1px dashed #999;
      cursor: pointer;
    }
  }
}
</style>
