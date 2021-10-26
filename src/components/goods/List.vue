<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加商品区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getGoodsList">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table表格区域 -->
      <el-table :data="goodsList"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称"
                         prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)"
                         prop="goods_price"
                         width="105px"></el-table-column>
        <el-table-column label="商品数量"
                         prop="goods_number"
                         width="80px"></el-table-column>
        <el-table-column label="商品重量"
                         prop="goods_weight"
                         width="80px"></el-table-column>
        <el-table-column label="创建时间"
                         width="160px">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="140px">
          <!-- eslint-disable -->
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="showEditDialog(scope.row.goods_id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeGoods(scope.row.goods_id)"></el-button>
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

    <!-- 编辑按钮 对话框 -->
    <el-dialog title="编辑商品"
               :visible.sync="editDialogVisible"
               width="50%">
      <el-form ref="editDialogRef"
               :model="editForm"
               label-width="110px"
               :rules="editRules">
        <el-form-item label="商品名称"
                      prop="goods_name">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格(元)"
                      prop="goods_price">
          <el-input v-model="editForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品数量"
                      prop="goods_number">
          <el-input v-model="editForm.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量"
                      prop="goods_weight">
          <el-input v-model="editForm.goods_weight"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="changeGoods">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表数据
      goodsList: [],
      // 总共商品条数
      total: 0,
      editDialogVisible: false,
      editForm: {},
      editRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
      }
    };
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 获取商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败!')
      }
      // this.$message.success('获取商品列表成功!')
      this.goodsList = res.data.goods
      this.total = res.data.total
      console.log(this.queryInfo.pagenum);

    },
    //每页条数
    handleSizeChange (newPage) {
      this.queryInfo.pagesize = newPage
      this.getGoodsList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },
    // 删除商品
    async removeGoods (goodsId) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('您已取消删除!')
      }

      const { data: res } = await this.$http.delete(`goods/${goodsId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败!')
      }
      this.$message.success('删除成功!')
      this.getGoodsList()
    },
    goAddpage () {
      this.$router.push('/goods/add')
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品信息失败!')
      }
      this.editForm = res.data
      console.log(this.editForm);

      this.editDialogVisible = true
    },
    changeGoods () {
      this.$refs.editDialogRef.validate(async valid => {
        if (!valid) {
          return
          // return this.$message.error('修改失败!')
        }
        // {
        //   goods_name: this.editForm.goods_name,
        //   goods_price: this.editForm.goods_price,
        //   goods_number: this.editForm.goods_number,
        //   goods_weight: this.editForm.goods_weight

        // }
        const { data: res } = await this.$http.put(`goods/${this.editForm.goods_id}`, this.editForm)
        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败!')
        }
        this.$message.success('修改成功!')

        this.editDialogVisible = false
        this.getGoodsList()
      })
    }
  },
  computed: {
    // goodId () {
    //   if (this.editForm.goods_id) {
    //     return this.editForm.goods_id
    //   }
    // }
  }
};
</script>

<style lang="less" scoped>
</style>