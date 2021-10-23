<template>
  <div>
    <el-button @click="logout" type="info">退出</el-button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  methods: {
    logout () {
      // 清除token
      window.sessionStorage.clear()
      // 重定向到登录页面
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
