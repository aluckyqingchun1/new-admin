<template>
  <section>
    <div class="title" :class="{ 'sub': subTitle }">
      <span>{{ title }}</span><span v-if="subTitle" class="sub-title">{{ subTitle }}</span>
    </div>
    <slot />
  </section>
</template>

<script>
export default {
  name: 'Box',
  props: {
    title: {
      type: String,
      default: '九慧开发平台'
    },
    subTitle: {
      type: String,
      default: ''
    }
  },
  data() {
    return {}
  },
  methods: {}
}
</script>

<style lang='scss' scoped>
section {
  width: 380px;
  padding: 0 40px;
  box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.1);
  border-radius: 4px;
  .title {
    text-align: center;
    padding:35px 0 28px 0;
    font-size: 20px;
    font-family: Microsoft YaHei;
    line-height: 49px;
    color: rgba(0, 0, 0, 0.85);

  }
}
</style>
