<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片区，用布局容器布局元素 -->
    <el-card>
      <el-row>
        <el-col :span="10">
          <div>
            <!-- 添加用户按钮 -->
            <el-button type="primary">添加用户</el-button>
          </div>
        </el-col>
      </el-row>
      <el-table :data="roleList" border stripe>
        <!-- 查看权限列表 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="item1 in scope.row.children" :key="item1.id">
              <el-col :span="4"
                ><el-tag> {{ item1.authName }}</el-tag
                ><i class="el-icon-caret-right"></i
              ></el-col>
              <el-col :span="20">
                <el-row v-for="item2 in item1.children" :key="item2.id">
                  <el-col :span="4">
                    <el-tag type="success"> {{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="20">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="delCurrentRight(scope.row, item3.id)"
                    >
                      {{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
              {{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- <pre> {{ scope.row }}</pre> -->
            <el-button type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete">删除当前</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              @click="showRightTree(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 角色分配对话框 -->
    <el-dialog
      title="角色分配"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="clearRightChecked"
    >
      <el-tree
        :data="treeData"
        default-expand-all
        :default-checked-keys="defsArray"
        show-checkbox
        :props="treeProps"
        node-key="id"
        ref="addRightRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="newAddRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      roleList: [],
      setRightDialogVisible: false,
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      treeData: [],
      defsArray: [],
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    // 获取角色列表，将所有角色对应信息渲染到页面上
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      console.log(res)
      if (res.meta.status !== 200) {
        return
      }
      this.roleList = res.data
    },
    // 删除三级权限，
    async delCurrentRight(role, rightId) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(
            `roles/${role.id}/rights/${rightId}`
          )
          // console.log(res)
          if (res.meta.status !== 200) {
            this.$remindMes({
              type: 'info',
              message: '权限删除失败'
            })
            return
          }
          role.children = res.data
          this.$remindMes({
            type: 'success',
            message: '权限删除成功!'
          })
        })
        .catch(() => {
          this.$remindMes({
            type: 'info',
            message: '已取消删除此权限'
          })
        })
    },
    // 点击分配权限按钮时出现分配权限对话框
    async showRightTree(role) {
      // alert(123)
      const { data: res } = await this.$http.get('rights/tree')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('分配权限请求失败')
      }
      this.treeData = res.data
      this.roleId = role.id
      this.getThirdRightId(role, this.defsArray)
      this.setRightDialogVisible = true
    },
    // 获取所有三级节点的id值☆☆☆☆☆☆☆☆
    getThirdRightId(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getThirdRightId(item, arr)
        // this.getThirdRightId(node, arr)
      })
    },
    // 分配权限对话框关闭后，清空当前角色拥有的权限
    clearRightChecked() {
      this.defsArray = []
    },
    // 点击对话框中的确定按钮时，为角色新增用户权限
    async newAddRight() {
      const arr1 = this.$refs.addRightRef.getHalfCheckedKeys()
      // console.log(arr1)
      const arr2 = this.$refs.addRightRef.getCheckedKeys()
      // console.log(arr2)
      const keys = [...arr1, ...arr2]
      // console.log(keys)
      const ids = keys.join(',')
      // console.log(ids)
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: ids }
      )
      if (res.meta.status !== 200) {
        this.setRightDialogVisible = false
        return this.$remindMes.error('新增权限失败')
      }
      this.setRightDialogVisible = false
      this.$remindMes.success('新增权限成功')
      this.getRoleList()
    }
  }
}
</script>
<style scoped></style>
