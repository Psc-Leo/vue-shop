<template>
    <div>
      <!-- 面包屑导航区域 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>订单管理</el-breadcrumb-item>
        <el-breadcrumb-item>订单列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图-->
      <el-card>
        <el-row>
          <el-col :span="8">
            <el-input placeholder="请输入内容">
              <template #append>
                <el-button icon="el-icon-search"></el-button>
              </template>
            </el-input>
          </el-col>
        </el-row>

        <!-- 订单列表数据 -->
        <el-table :data="orderList" stripe border>
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="order_number" label="订单编号"></el-table-column>
          <el-table-column label="订单价格" prop="order_price"></el-table-column>
          <el-table-column label="是否付款" prop="pay_status">
            <template slot-scope="scope">
              <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
              <el-tag type="danger" v-else>未付款</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="是否发货" prop="is_send">
            <template slot-scope="scope">
              <template>
                {{scope.row.is_send}}
              </template>
            </template>
          </el-table-column>
          <el-table-column label="下单时间" prop="create_time">
            <template slot-scope="scope">
              {{scope.row.create_time | dateFormat}}
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template>
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
              <el-button type="success" icon="el-icon-location
" size="mini" @click="showProgressBox"></el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页区域 -->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[2, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total"></el-pagination>

      </el-card>

      <!--修改地址的对话框-->
      <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="handleChange" >
        <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormForm" label-width="100px" >
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader v-model="addressForm.address1" :options="cityData" :props="orderProps" clearable></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="addressVisible = false">取 消</el-button>
            <el-button type="primary" @click="addressVisible = false">确 定</el-button>
          </span>
        </template>
      </el-dialog>

      <!-- 展示物流进度的对话框 -->
      <el-dialog
        title="物流进度"
        :visible.sync="progressVisible"
        width="50%"
      >
        <!-- 时间线 -->
        <el-timeline>
          <el-timeline-item
            v-for="(activity, index) in this.progressInfo"
            :key="index"
            :timestamp="activity.time">
            {{activity.context}}
          </el-timeline-item>
        </el-timeline>
      </el-dialog>
    </div>
</template>

<script>
  // 导入省市区/县数据
  import cityData from './citydata.js'

  export default {
    data() {
      return {
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 10
        },
        total: 0,
        orderList: [],
        // 修改地址的对话框的显示与隐藏
        addressVisible: false,
        // 表单数据对象
        addressForm: {
          address1: [],
          address2: ''
        },
        addressFormRules: {
          address1: [
            { required: true, message: '请选择省市区县', trigger: 'blur' }
          ],
          address2: [
            { required: true, message: '请填写详细地址', trigger: 'blur' }
          ]
        },
        cityData,
        // 级联选择框的配置对象
        orderProps: {
          expandTrigger: 'hover'
        },
        progressVisible: false,
        progressInfo: []
      }
    },
    created() {
      this.getOrderList()
    },
    methods: {
      async getOrderList() {
        const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
        if (res.meta.status !== 200) return this.$message.error('获取订单列表失败！')
        // console.log(res)
        this.total = res.data.total
        this.orderList = res.data.goods
      },
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getOrderList()
      },
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getOrderList()
      },
      // 展示修改地址的对话框
      showBox() {
        this.addressVisible = true
      },
      // 关闭对话框，清除内容
      handleChange() {
        this.$refs.addressFormForm.resetFields()
      },
      // 物流进度的对话框
      async showProgressBox() {
        const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
        if (res.meta.status !== 200) {
          return this.$message.error('获取物流进度失败！')
        }

        this.progressInfo = res.data

        this.progressVisible = true
        console.log(this.progressInfo)
      }
    }
  }
</script>

<style lang="less" scoped>
  .el-cascader {
    width: 100%;
  }
</style>
