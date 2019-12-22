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
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row class="bdtop vcenter" v-for="item1 in scope.row.children" :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag @close="removeRightById(scope.row, item1.id)" closable>{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限和三级 -->
              <el-col :span="19">
                <el-row class="bdbottom,bdtop,vcenter" v-for="item2 in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" @close="removeRightById(scope.row, item2.id)" closable>
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" @close="removeRightById(scope.row, item3.id)" closable class="[bdbottom,bdtop]" v-for="item3 in item2.children" :key="item3.id">
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
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
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--  添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDialogClosed" width="40%">
      <!--内容主体 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="90px">
        <el-form-item label="角色名称" prop="rolesName">
          <el-input v-model="addForm.rolesName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!--  分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" @close="setRightDialogClosed" width="50%">
      <!--内容主体 树形控件 -->
      <el-tree :data="rightslist" ref="treeRef" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" :props="treeProps"></el-tree>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 验证角色的规则
    let checkrolesName = (rule, value, cb) => {
      // 验证邮箱的正则
      const regrolesName = /^[\u4e00-\u9fa5]{3,10}$/
      if (regrolesName.test(value)) {
        // 合法邮箱
        return cb()
      }
      cb(new Error('请输入合法角色名'))
    }
    return {
      // 所有角色列表数据
      rolelist: [],
      //  控制添加对话框显示和隐藏
      addDialogVisible: false,
      //   添加用户的数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 所有权限的数据
      rightslist: [],
      // 树形控件的数据对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 控制分配权限的对话框显示与隐藏
      setRightDialogVisible: false,
      // 默认选中的节点id
      defKeys: [],
      //  当前即将分配权限的id
      roleId: '',
      //   添加表单的验证规则对象
      addFormRules: {
        rolesname: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 3, max: 10, message: '角色名称长度在3-10个字符', trigger: 'blur' },
          { validator: checkrolesName, trigger: 'blur' }
        ]
      }
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
    },
    // 监听角色添加对话框的关闭事件
    addDialogClosed() {
      //  关闭添加对话框重置表单内容
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加角色
    addRoles() {
      this.$refs.addFormRef.validate(async valid => {
        //  console.log(valid)
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功!')
        // 隐藏添加的用户的对话框
        this.addDialogVisible = false
        // 重新获取角色列表数据
        this.getRolesList()
      })
    },
    //   根据id删除对应的权限
    async removeRightById(role, rightId) {
      //  弹框提示永久删除该文件 是否要继续
      const confirmResult = await this.$confirm('此文件永久删除是否要继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除!')
      }
      //  console.log('确认删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.info('删除权限失败')
      }
      role.children = res.data
      this.$message.success('删除成功')
    },
    // 展示权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message('获取权限失败')
      }
      // 获取到的权限数据 保存到data中
      this.rightslist = res.data
      //  递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式 获取角色下所有的三级权限的id,并保存到defKey 数组中
    getLeafKeys(node, arr) {
      //  如果单签 node节点不包含children数据 则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的显示 关闭清空默认勾选数据
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]

      // console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更新权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-top: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
