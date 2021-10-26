<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getUserList">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名"
                         prop="username"></el-table-column>
        <el-table-column label="邮箱"
                         prop="email"></el-table-column>
        <el-table-column label="电话"
                         prop="mobile"></el-table-column>
        <el-table-column label="角色"
                         prop="role_name"></el-table-column>
        <el-table-column label="状态"
                         prop="mg_state">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state"
                       @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="220px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="changeUser(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeUser(scope.row.id)"></el-button>
            <!-- 分配按钮 -->
            <el-tooltip effect="dark"
                        content="分配角色"
                        placement="top"
                        :enterable="false">
              <el-button type="warning"
                         icon="el-icon-setting"
                         size="mini"
                         @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[1, 2, 5, 10]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="70px">
        <el-form-item label="用户名"
                      prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码"
                      prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱"
                      prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机"
                      prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog title="修改用户"
               :visible.sync="changeDialogVisible"
               width="50%"
               @close="changeDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="changeForm"
               :rules="changeFormRules"
               ref="changeFormRef"
               label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="changeForm.username"
                    disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱"
                      prop="email">
          <el-input v-model="changeForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机"
                      prop="mobile">
          <el-input v-model="changeForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="changeDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="changeUsers">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色的对话框 -->
    <el-dialog title="分配角色"
               :visible.sync="setRoleDialogVisible"
               width="50%"
               @close="setRoleDialogClosed">
      <el-form :model="userinfo"
               label-width="100px">
        <el-form-item label="当前的用户">
          <el-input v-model="userinfo.username"
                    disabled></el-input>
        </el-form-item>
        <el-form-item label="当前的角色">
          <el-input v-model="userinfo.role_name"
                    disabled></el-input>
        </el-form-item>
        <el-form-item label="分配新角色">
          <el-select v-model="selectRoleId"
                     placeholder="请选择">
            <el-option v-for="item in rolesList"
                       :key="item.id"
                       :label="item.roleName"
                       :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    //验证邮箱规则
    var checkEmail = (rule, value, callback) => {
      //验证邮箱正则
      const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    };
    //验证手机号规则
    var checkMobile = (rule, value, callback) => {
      //验证手机号正则
      const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    };
    return {
      //获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      //获取当前行用户的列表
      userinfo: {},
      //获取所有角色的数据列表
      rolesList: [],
      // 被选中的角色id
      selectRoleId: '',
      //控制添加对话框显示与隐藏
      addDialogVisible: false,
      //控制修改对话框显示与隐藏
      changeDialogVisible: false,
      //控制分配权限对话框显示与隐藏
      setRoleDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      //修改的用户信息
      changeForm: {},
      //添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: "请输入邮箱", trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: "请输入手机号", trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
      },
      //修改表单的验证规则对象
      changeFormRules: {
        email: [
          { required: true, message: "请输入邮箱", trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: "请输入手机号", trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 监听handleSizeChange改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    //监听页码值改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    //监听 switch 状态的改变
    async userStateChange (userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      // console.log(res);
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    //监听添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮，添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async vaild => {
        if (!vaild) return
        //可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        // const { data: res } = await this.$http.get('personalized')
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')

        //隐藏添加用户的对话框
        this.addDialogVisible = false

        //重新获取用户列表数据
        this.getUserList()
      })
    },
    //展示编辑用户的对话框
    async changeUser (id) {
      const { data: res } = await this.$http.get('users/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('修改用户失败')
      }
      this.changeForm = res.data
      this.changeDialogVisible = true
    },
    //监听修改用户对话框的关闭事件
    changeDialogClosed () {
      this.$refs.changeFormRef.resetFields()
    },
    //点击按钮，修改用户
    changeUsers () {
      this.$refs.changeFormRef.validate(async vaild => {
        if (!vaild) return
        //可以发起修改用户的网络请求
        const { data: res } = await this.$http.put('users/' + this.changeForm.id, { email: this.changeForm.email, mobile: this.changeForm.mobile })

        if (res.meta.status !== 200) {
          return this.$message.error('修改用户信息失败')
        }
        this.$message.success('修改用户信息成功')

        //隐藏修改用户的对话框
        this.changeDialogVisible = false

        //重新获取用户列表数据
        this.getUserList()
      })
    },
    //根据id删除用户
    async removeUser (id) {
      const confirmRes = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmRes !== 'confirm') {
        return this.$message('已取消删除')
      }

      const { data: res } = await this.$http.delete('users/' + id, { id })

      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')

      //重新获取用户列表数据
      this.getUserList()
    },
    // 分配角色按钮事件
    async setRole (userinfo) {
      this.userinfo = userinfo

      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    //点击按钮，分配角色
    async saveRoleInfo () {
      if (!this.selectRoleId) {
        return this.$message.error('请选择要分配的角色！')
      }
      const { data: res } = await this.$http.put(`users/${this.userinfo.id}/role`, { rid: this.selectRoleId })
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色失败！')
      }
      this.$message.success('更新角色成功！')

      this.getUserList()
      this.setRoleDialogVisible = false
    },
    //监听分配角色狂的关闭事件 
    setRoleDialogClosed () {
      this.selectRoleId = ''
      this.userinfo = {}
    }
  }
};
</script>

<style lang="less" scoped>
.el-select {
  width: 100%;
}
</style>