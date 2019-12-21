<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!--  添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
          <!-- 展开列 -->
        <el-table-column type="expand"></el-table-column>
          <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="">
            <el-button
              size="mini"
              type="primary"
              icon="iconfont icon-bianji
"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="iconfont icon-shanchu
"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="iconfont icon-shezhi
"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolelist: []
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表数据
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }
      this.rolelist = res.data
      // console.log(this.rolelist)
    }
  }
}
</script>

<style lang="less" scoped></style>
