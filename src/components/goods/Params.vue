<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- 顶部 警告区域 -->
      <el-alert title="注意"
                type="warning"
                description="只允许为第三级分类设置相关参数"
                show-icon
                :closable="false">
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row>
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader v-model="selectedKeys"
                       :options="cateList"
                       :props="cateListProps"
                       @change="cateChange"
                       style="width:200px"></el-cascader>
        </el-col>
      </el-row>
      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <!-- 动态参数的控制面板 -->
        <el-tab-pane label="动态参数"
                     name="many">
          <!-- 添加参数按钮 -->
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">
            添加参数
          </el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTabData"
                    border
                    stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染 tag 标签 -->
                <el-tag v-for="(item,i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClose(i,scope.row)">{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          style="width:150px"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 切换 tag 的按钮 -->
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- 修改按钮 -->
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                <!-- 删除按钮 -->
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性的控制面板 -->
        <el-tab-pane label="静态属性"
                     name="only">
          <!-- 添加属性按钮 -->
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">
            添加属性
          </el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTabData"
                    border
                    stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染 tag 标签 -->
                <el-tag v-for="(item,i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClose(i,scope.row)">{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          style="width:150px"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 切换 tag 的按钮 -->
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- 修改按钮 -->
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                <!-- 删除按钮 -->
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加动态参数和静态属性对话框 -->
    <el-dialog :title="'添加' + titleText"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <!-- 添加参数和静态属性表单 -->
      <el-form :model="addForm"
               :rules="addRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="manyDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数的对话框 -->
    <el-dialog :title="'修改' + titleText"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClosed">
      <!-- 修改参数和静态属性表单 -->
      <el-form :model="editForm"
               :rules="editRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 商品分类数据
      cateList: [],
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 指定 级联选择器 的配置对象
      cateListProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      //选中的数组 id
      selectedKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数的数据
      manyTabData: [],
      // 静态属性的数据
      onlyTabData: [],
      // 添加动态参数对话框的显示与隐藏
      addDialogVisible: false,
      // 添加参数的表单对象
      addForm: {
        attr_name: ''
      },
      // 添加表单的验证规则
      addRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 修改参数的对话框 显示与隐藏
      editDialogVisible: false,
      // 修改的表单数据
      editForm: {},
      // 修改表单的验证规则
      editRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 控制 输入框 与 按钮 的切换显示
      inputVisible: false,
      // 输入框的值
      inputValue: ''
    };
  },
  created () {
    //调用 获取商品分类数据 的函数
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.cateList = res.data
    },
    // 选择项 发生变化 触发函数
    cateChange () {
      this.getParamsData()
    },
    // 获取参数的列表数据
    async getParamsData () {
      // 证明选中的不是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTabData = []
        this.onlyTabData = []
        this.$message.warning('请选择三级分类！')
        return
      }
      //根据所选分类的Id 和 当前所处的面板 来获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败！')
      }

      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制 输入框 与 按钮 的切换显示
        item.inputVisible = false
        // 输入框的值
        item.inputValue = ''
      })
      console.log(res.data);

      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
    },
    // 页签的默认点击事件
    handleTabClick () {
      console.log(this.activeName);
      this.getParamsData()
    },
    // 监听对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮 添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return

        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 修改参数的点击的事件
    async showEditDialog (attr_id) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 关闭对话框 清空选项 重置表单
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮 修改参数
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return

        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 200) {
          return this.$message.error('参数更新失败！')
        }

        this.$message.success('参数更新成功！')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除参数
    async removeParams (attr_id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      this.$message.success('删除成功！')
      this.getParamsData()

    },
    // 文本框失去焦点 或者 按下 enter 都会触发
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }

      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false

      this.saveAttrVals(row)
    },
    // 将对 attr_vals 的操作保存到数据库
    async saveAttrVals (row) {
      // 发起请求,保存数据
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, { attr_name: row.attr_name, attr_sel: row.attr_sel, attr_vals: row.attr_vals.join(' ') })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数失败!')
      }
      this.$message.success('修改参数成功!')
    },
    // 显示输入框
    showInput (row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回 true ，否则返回 false
    isBtnDisabled () {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类id
    cateId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    // 动态计算标题文本
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
};
</script>

<style lang="less" scoped>
.el-alert {
  margin-bottom: 10px;
}
.el-tabs {
  margin-top: 10px;
}
.el-tag {
  margin: 6px;
}
</style>