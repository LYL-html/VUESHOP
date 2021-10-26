<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 顶部搜索框区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append"
                       icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <el-table :data="orderList"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number"
                         label="订单编号"
                         width="">
        </el-table-column>
        <el-table-column prop="order_price"
                         label="订单价格"
                         width="80px">
        </el-table-column>
        <el-table-column label="是否付款"
                         width="80px">
          <template slot-scope="scope">
            <el-tag type="success"
                    v-if="scope.row.pay_status === '1'"
                    size="mini">已付款</el-tag>
            <el-tag type="danger"
                    size="mini"
                    v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send"
                         label="是否发货"
                         width="80px">
        </el-table-column>
        <el-table-column label="下单时间"
                         width="180px">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="150px">
          <template>
            <!-- 编辑按钮 -->
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="showAddress"></el-button>
            <el-button type="success"
                       icon="el-icon-location-outline"
                       size="mini"
                       @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[5, 10, 12, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>

    <!-- 修改地址对话框 -->
    <el-dialog title="修改地址"
               :visible.sync="addressDialogVisible"
               width="50%"
               @close="addressClosed">
      <!-- 表单区域 -->
      <el-form :model="addressForm"
               :rules="addressFormRules"
               ref="addressFormRef"
               label-width="100px">
        <el-form-item label="省市区/县"
                      prop="address1">
          <!-- 级联选择器 -->
          <el-cascader :options="cityOptions"
                       v-model="addressForm.address1"
                       :props="{ expandTrigger: 'hover' }"
                       @change="changeProvince">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址"
                      prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 展示物流进度对话框 -->
    <el-dialog title="提示"
               :visible.sync="progressVisible"
               width="50%">
      <!-- 时间线 -->
      <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressList"
                          :key="index"
                          :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityOptions from './city_data2017_element.js'
import progressInfo from './progressInfo.json'

export default {
  data () {
    return {
      // 商品列表数据
      orderList: [],
      // 商品查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      pagenum: 0,
      addressDialogVisible: false,
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请选择详细地址', trigger: 'blur' }
        ]
      },
      addressForm: {
        address1: [],
        address2: ''
      },
      cityOptions,
      progressVisible: false,
      progressList: progressInfo.data
    };
  },
  created () {
    this.getOrderList()
  },
  methods: {
    // 获取订单列表数据
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败!')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    //每页条数
    handleSizeChange (newPage) {
      this.queryInfo.pagesize = newPage
      this.getOrderList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getOrderList()
    },
    // 点击按钮 显示修改地址对话框
    showAddress () {
      this.addressDialogVisible = true
    },
    changeProvince () { },
    addressClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox () {
      console.log(this.progressList)


      this.progressVisible = true
    }
  },
};
</script>

<style lang="less" scoped>
@import '../../plugins/timeline/timeline.css';
@import '../../plugins/timeline-item/timeline-item.css';
.el-cascader {
  width: 100%;
}
</style>