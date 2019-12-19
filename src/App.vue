<template>
  <div id="app">
        <el-page-header @back="goBack" content="详情页面"> </el-page-header>

    <el-button plain icon autofocus>朴素按钮</el-button>
      <el-button type="primary" plain >主要按钮</el-button>
      <el-button type="success" plain >成功按钮</el-button>
      <el-button type="info" plain >信息按钮</el-button>
      <el-button type="warning" plain >警告按钮</el-button>
      <el-button type="danger" plain >危险按钮</el-button>
  </div>
</template>

<script>

export default {
  name: 'app',
  components: {},
  methods: {
    goBack () {
      console.log('go back')
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
