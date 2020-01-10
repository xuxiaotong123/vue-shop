<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片区，用布局容器布局元素 -->
    <el-card>
      <el-row :gutter="16">
        <el-col :span="10">
          <div>
            <!-- 搜索框 -->
            <el-input placeholder="请输入内容" class="input-with-select" clearable v-model="queryInfo.query">
              <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="5">
          <div>
            <!-- 添加用户按钮 -->
            <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
          </div>
        </el-col>
      </el-row>
      <!-- 用户列表区 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope="scope">
            <!-- {{ scope.row }} -->
            <!-- 用户状态开关 -->
            <el-switch v-model="scope.row.mg_state" @change="statusChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <!-- {{ scope.row }} -->
            <el-button type="primary" icon="el-icon-edit" circle @click="editUser(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" circle @click="showDelDialog(scope.row.id)"></el-button>
            <!-- 角色管理按钮 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-star-off" circle @click="showChangeRoleDialog(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="clearAddContent">
      <!-- 添加用户对话框的form表单 -->
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="用户密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUserSure">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户对话框 -->
    <el-dialog title="编辑用户" :visible.sync="editDialogVisible" width="50%" @close="clearEditContent">
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" disabled=""></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserSure()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="changeRoleDialog" width="30%" @close="clearCheckedRole">
      <div>
        当前的用户：<span>{{ userInfo.username }}</span>
      </div>
      <div>
        当前的角色：<span>{{ userInfo.role_name }}</span>
      </div>
      <div>
        分配新角色：<el-select v-model="selectedRole" placeholder="请选择">
          <el-option v-for="item in allRoleslist" :key="item.id" :label="item.roleName" :value="item.id">{{ item.roleName }} </el-option>
        </el-select>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="changeRoleDialog = false">取 消</el-button>
        <el-button type="primary" @click="sureNewRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    // var checkEmail = (rule, value, callback) => {}
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return callback()
      } else {
        callback(new Error('请输入合法的邮箱'))
      }
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {},
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, max: 10, message: '长度在 5 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }],
        mobile: [
          { required: true, message: '请输入活动手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }],
        mobile: [
          { required: true, message: '请输入活动手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      changeRoleDialog: false,
      userInfo: {},
      allRoleslist: [],
      selectedRole: ''
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    // 发请求获取数据，将数据渲染到页面上
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      // console.log(res)
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 更改当前页面显示数据的条数
    handleSizeChange(newPageSize) {
      // console.log(newPageSize)
      this.queryInfo.pagesize = newPageSize
      this.getUserList()
    },
    // 更改当前页为第几页
    handleCurrentChange(newPageSize) {
      // console.log(newPageSize)
      this.queryInfo.pagenum = newPageSize
      this.getUserList()
    },
    // 用户状态修改（是否为激活状态）
    async statusChanged(usersState) {
      const { data: res } = await this.$http.put(`users/${usersState.id}/state/${usersState.mg_state}`)
      console.log(res)
      if (res.meta.status !== 200) {
        // 如果数据库中没有修改成功，但是页面上点击之后状态已经发生了改变 ，就会导致页面与数据库信息不一致，所以要将页面上的状态值重置，保证页面与数据库状态一致。
        usersState.mg_state = !usersState.mg_state
        return this.$remindMes.error('用户状态修改失败')
      }
      this.$remindMes.success('用户状态修改成功')
    },
    // 关闭(包括确定按钮的点击，点击确定时需要提交 ，需要另行添加事件)添加对话框时自动重置表单，清除验证规则
    clearAddContent() {
      this.$refs.addFormRef.resetFields()
    },
    clearEditContent() {
      this.$refs.editFormRef.resetFields()
    },
    // 表单添加预验证
    addUserSure() {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post('users', this.addForm)
        // console.log(res)
        if (res.meta.status !== 200) {
          this.$remindMes.error('用户添加失败')
        }
        this.$remindMes.success('用户添加成功')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    async editUser(id) {
      console.log(id)
      const { data: res } = await this.$http.get(`users/${id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('用户查询失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editUserSure() {
      // console.log(this.editForm.id)
      // 确认提交前要进行预验证,预验证之前通过
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$remindMes.error('用户更新失败')
        }
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    showDelDialog(id) {
      // console.log(id)
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`users/${id}`)
          // console.log(res)
          if (res.meta.status !== 200) {
            this.$remindMes({
              type: 'info',
              message: '删除用户失败'
            })
            return
          }
          this.getUserList()
          this.$remindMes({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$remindMes({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 点击分配角色按钮 弹出分配角色对话框
    async showChangeRoleDialog(userInfo) {
      this.userInfo = userInfo
      const { data: res } = await this.$http.get('roles')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('分配角色失败')
      }
      this.allRoleslist = res.data
      this.changeRoleDialog = true
    },
    clearCheckedRole() {
      this.selectedRole = ''
    },
    async sureNewRole() {
      if (this.selectedRole === '') {
        return this.$remindMes.error('请选择要新分配的权限')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selectedRole
      })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error(res.meta.msg)
      }
      this.changeRoleDialog = false
      this.getUserList()
      this.$remindMes.success('设置角色成功')
    }
  }
}
</script>
<style lang="less" scoped></style>
