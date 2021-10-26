<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="addSort">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table :data="cateList"
                  :columns="columns"
                  :selection-type="false"
                  :expand-type="false"
                  border
                  show-index
                  index-text="#"
                  class="terrTable">
        <!-- 是否有效 模板-->
        <template slot="isok"
                  slot-scope="scope">
          <i class="el-icon-success"
             style="color:#45b97c"
             v-if="!scope.row.cat_deleted"></i>
          <i class="el-icon-error"
             style="color:red"
             v-else></i>
        </template>
        <!-- 排序 模板-->
        <template slot="order"
                  slot-scope="scope">
          <!-- 一级 -->
          <el-tag v-if="scope.row.cat_level === 0"
                  size="mini">一级</el-tag>
          <!-- 二级 -->
          <el-tag v-else-if="scope.row.cat_level === 1"
                  type="success"
                  size="mini">二级</el-tag>
          <!-- 三级 -->
          <el-tag v-else
                  type="warning"
                  size="mini">三级</el-tag>
        </template>
        <!-- 操作 模板 -->
        <!-- eslint-disable -->
        <template slot="operate"
                  slot-scope="scope">
          <el-button type="primary"
                     icon="el-icon-edit"
                     size="mini"
                     @click="editCate(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger"
                     icon="el-icon-delete"
                     size="mini"
                     @click="removeCates(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 10, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类"
               :visible.sync="addSortDialogVisible"
               width="50%"
               @close="addSortDialogClosed">
      <el-form :model="addCateForm"
               :rules="addCateFormRules"
               ref="addCateFormRef"
               label-width="100px">
        <!-- 分类名称输入框 -->
        <el-form-item label="分类名称"
                      prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <!-- 分类名称下拉框 -->
        <el-form-item label="父级分类">
          <!-- options 用来指定数据源 -->
          <!-- props 用来指定配置对象-->
          <el-cascader v-model="selectedKeys"
                       :options="parentCateList"
                       :props="cascaderProps"
                       @change="parentCateChange"
                       expandTrigger="hover"
                       style="width:200px"
                       clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addSortDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色对话框 -->
    <el-dialog title="编辑角色"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editClosed">
      <!-- 表单区域 -->
      <el-form :model="editForm"
               :rules="editRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item label="分类名称"
                      prop="cat_name">
          <el-input v-model="editForm.cat_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="changeCates">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {

    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      //父级分类列表p
      parentCateList: [],
      // 总数居条数
      total: 0,
      //添加分类对话框 显示与隐藏
      addSortDialogVisible: false,
      //为table指定列的定义
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        // 将当前列定义为模板列
        type: 'template',
        // 表示当前列使用的模板名称
        template: 'isok'
      }, {
        label: '排序',
        // 将当前列定义为模板列
        type: 'template',
        // 表示当前列使用的模板名称
        template: 'order'
      }, {
        label: '操作',
        // 将当前列定义为模板列
        type: 'template',
        // 表示当前列使用的模板名称
        template: 'operate'
      }],

      //添加分类的表单 对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 将要添加的 分类父 ID
        cat_pid: 0,
        // 将要添加的 分类层级
        cat_level: 0
      },
      // 添加分类对话框 规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 指定 级联选择器 的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
      },
      //选中的父级数组 id
      selectedKeys: [],
      editForm: {},
      editDialogVisible: false,
      editRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
      }
    };
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      // 把数据列表赋值给 cateList
      this.cateList = res.data.result
      // 将总数居条数分配给 total
      this.total = res.data.total
      // console.log(this.cateList);
      // console.log(this.total);
    },
    // 监听 pagesize 改变
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      //立即刷新商品分类数据
      this.getCateList()
    },
    //监听 pagenum 改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      //立即刷新商品分类数据
      this.getCateList()
    },
    //点击按钮 添加分类
    addSort () {
      //先 获取父级分类的数据列表
      this.getParentCateList()
      //点击添加按钮 分类对话框 显示
      this.addSortDialogVisible = true
    },
    //获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.parentCateList = res.data
      console.log();
    },
    // 选择项 发生变化 触发函数
    parentCateChange () {
      //如果 selectedKeys 数组中的 length大于0，证明选中父级分类
      // 反之 就没有选中 父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        // 父级分类id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    //点击按按钮 添加分类
    addCate () {
      // console.log(this.addCateForm);
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addSortDialogVisible = false
      })
    },
    //监听添加分类对话框关闭事件,重置表单数据
    addSortDialogClosed () {
      // 重置表单
      this.$refs.addCateFormRef.resetFields()
      // 重置 selectedKeys 数组
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    async editCate (id) {
      const { data: res } = await this.$http.get(`categories/${id}`)

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品信息失败!')
      }
      this.editForm = res.data
      console.log(this.editForm);

      this.editDialogVisible = true

    },
    changeCates () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`categories/${this.editForm.cat_id}`, { cat_name: this.editForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败!')
        }
        this.$message.success('修改成功!')

        this.editDialogVisible = false
        this.getCateList()
        // this.getParentCateList()
      })
    },
    // 删除分类
    async removeCates (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('您已取消删除!')
      }

      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败!')
      }
      this.$message.success('删除成功!')
      this.getCateList()
    },
    editClosed () {
      this.$refs.editFormRef.resetFields()
    }

  }
};
</script>

<style lang="less" scoped>
.terrTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100% !important;
}
</style>