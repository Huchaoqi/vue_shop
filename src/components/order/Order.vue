<template>
  <div>
    <!-- 面包屑导航 -->
    <bd nav1="订单管理" nav2="订单列表"></bd>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <!--搜索文本输入框  -->
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!--  订单数据表格 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number">
          <template slot-scope="scope">
            {{ scope.row.order_number }}
          </template>
        </el-table-column>
        <el-table-column label="订单价格" prop="order_price">
          <template slot-scope="scope">
            {{ scope.row.order_price }}
          </template>
        </el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else-if="scope.row.pay_status === '0'">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send">
          <template slot-scope="scope">
            {{ scope.row.is_send }}
          </template>
        </el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="showBox"></el-button>
          <el-button type="success" size="mini" icon="el-icon-location" @click="showProgressBox"></el-button>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 8, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total,sizes,prev,pager,next,jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址的对话框 -->
    <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="addressDialogClosed">
      <!-- 修改验证表单 -->
      <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <!-- <el-input v-model="addressForm.address1"></el-input> -->
          <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 展示物流进度的对话框 -->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%" @close="progressDialogClosed">
      <!-- 时间线 -->
      <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressInfo" :key="index" :timestamp="activity.time">
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      //  默认总条数
      total: 0,
      //  添加订单数据
      orderList: [],
      // 修改地址对话框的隐藏和显示
      addressVisible: false,
      // 要修改的数据表单
      addressForm: {
        address1: [],
        address2: ''
      },
      // 修改地址表单的验证规则
      addressFormRules: {
        address1: [{ required: true, message: '请选择省市区', trigger: 'blur' }],
        address: [{ required: true, message: '请填写省市区', trigger: 'blur' }]
      },
      cityData: cityData,
      //   控制物流进度对话框的显示和隐藏
      progressVisible: false,
      // 物流信息数据
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败!')
      }
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    // 分页发生变化
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getOrderList()
    },
    // 当前分页发生变化时
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 展示修改地址的对话框
    showBox() {
      // 点击按钮弹出修改对话框
      this.addressVisible = true
    },
    // 监听修改对话框关闭事件
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    // 点击显示物流对话框事件
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败')
      }
      this.progressInfo = res.data
      this.progressVisible = true
      console.log(this.progressInfo)
    },
    // 监听物流对话框关闭事件
    progressDialogClosed() {}
  }
}
</script>

<style lang="less" scoped>
@import '../../plugins/timeline/timeline.css';
@import '../../plugins/timeline-item/timeline-item.css';
.el-cascader {
  width: 100%;
}
</style>
