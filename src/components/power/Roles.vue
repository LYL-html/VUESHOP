<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="addRoleDialogVisible = true">添加按钮</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist"
                border
                stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- eslint-disable -->
            <el-row v-for="(item1,i1) in scope.row.children"
                    :key="item1.id"
                    class="col1"
                    :class="['bottom-border',i1 === 0 ?'top-border':'']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                        @close="removeRight(scope.row,item1.id)">{{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过 for 循环 嵌套渲染权限 -->
                <el-row v-for="(item2,i2) in item1.children"
                        :key="item2.id"
                        class="col2"
                        :class="[i2 === 0 ?'':'top-border']">
                  <!-- 渲染二级权限 -->
                  <el-col :span="5">
                    <el-tag closable
                            type="success"
                            @close="removeRight(scope.row,item2.id)">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="19">
                    <el-tag closable
                            type="warning"
                            v-for="(item3,i3) in item2.children"
                            :key="item3.id"
                            @close="removeRight(scope.row,item3.id)">{{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName"
                         label="角色名称">
        </el-table-column>
        <el-table-column prop="roleDesc"
                         label="角色描述">
        </el-table-column>
        <el-table-column prop="address"
                         label="操作"
                         width="300px">
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="editRoles(scope.row.id)">编辑</el-button>
            <el-button type="danger "
                       icon="el-icon-share"
                       size="mini"
                       @click="removeRoles(scope.row.id)">删除</el-button>
            <el-button type="warning "
                       icon="el-icon-setting"
                       size="mini"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addRoleDialogVisible"
               width="50%"
               @close="addRoleDialogClosed">
      <el-form :model="addRoleForm"
               :rules="addRoleFormRules"
               ref="addRoleFormRef"
               label-width="90px">
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限"
               :visible.sync="showSetRightDialogVisible"
               width="50%"
               @close="setRightDialogClosed">
      <!-- 树形权限 -->
      <el-tree :data="rightslist"
               :props="treeProps"
               show-checkbox
               default-expand-all
               node-key="id"
               :default-checked-keys="defKey"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="showSetRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRight">确 定</el-button>
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
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="changeRoles">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      //所有角色的数据
      rolelist: [],
      // 所有权限的数据
      rightslist: [],
      //当前即将被分配权限的id
      roleId: '',
      //树形控件的属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      //默认被选中的节点id数组
      defKey: [],
      addRoleForm: {
        roleName: "",
        roleDesc: ""
      },
      //添加角色对话框显示与隐藏
      addRoleDialogVisible: false,
      //分配权限对话框显示与隐藏
      showSetRightDialogVisible: false,
      // 编辑角色
      editDialogVisible: false,
      //添加表单的验证规则对象
      addRoleFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      editForm: {},

      editRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      }
    };
  },
  created () {
    this.getRoleList()
  },
  methods: {
    //获取所有角色的列表
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    //监听添加用户对话框的关闭事件
    addRoleDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
    },
    //点击按钮，显示 添加角色 对话框
    addRoles () {
      this.$refs.addRoleFormRef.validate(async vaild => {
        if (!vaild) return
        //可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addRoleForm)

        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')

        //隐藏添加角色的对话框
        this.addRoleDialogVisible = false

        //重新获取角色列表数据
        this.getRoleList()
      })
    },
    // 删除权限功能
    async removeRight (role, rightId) {
      const rightRes = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (rightRes !== 'confirm') {
        return this.$message.info('取消了删除')
      }

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.info('删除权限失败')
      }

      role.children = res.data
    },
    //点击按钮 显示 分配权限 对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      //获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rightslist = res.data
      this.getLeafKeys(role, this.defKey)


      this.showSetRightDialogVisible = true
    },
    //  通过递归的方式 来获取角色下所有三级权限的id 并保存到 defKey 数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })

    },
    //监听 分配权限对话框 的关闭事件
    setRightDialogClosed () {
      this.defKey = []
    },
    //点击确定按钮 为角色分配权限
    async allotRight () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      //隐藏分配权限的对话框
      this.showSetRightDialogVisible = false

      //重新获取权限列表数据
      this.getRoleList()
    },
    async editRoles (id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败!')
      }
      this.editForm = res.data
      console.log(this.editForm);

      this.editDialogVisible = true
    },
    // 点击按钮,编辑角色
    async changeRoles () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`roles/${this.editForm.roleId}`, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败!')
        }
        this.$message.success('修改成功!')

        this.editDialogVisible = false
        this.getRoleList()
      })
    },
    // 删除角色
    async removeRoles (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('您已取消删除!')
      }

      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败!')
      }
      this.$message.success('删除成功!')
      this.getRoleList()
    },
    editClosed () {
      this.$refs.editFormRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px;
}
.top-border {
  border-top: 1px solid #eee;
}
.bottom-border {
  border-bottom: 1px solid #eee;
}
.col1 {
  display: flex;
  align-items: center;
}
.col2 {
  display: flex;
  align-items: center;
}
</style>