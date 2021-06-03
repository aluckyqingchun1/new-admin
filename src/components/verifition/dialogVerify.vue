<template>
  <el-dialog
    :visible.sync="dialogVisible"
    width="30%"
    :before-close="handleClose"
  >
    <verification-code
      ref="verify"
      :mode="'pop'"
      :captcha-type="'blockPuzzle'"
      :img-size="{ width: '330px', height: '155px' }"
      @success="success"
    />
  </el-dialog>
</template>

<script>
export default {
  components: {
    VerificationCode: import('@/components/verifition/Verify.vue')
  },
  data() {
    return {
      dialogVisible: false
    }
  },
  methods: {}
}
</script>

<style lang="scss" scoped>
</style>
