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
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="authName" label="权限名称"></el-table-column>
        <el-table-column prop="path" label="路径"></el-table-column>
        <el-table-column label="等级">
          <template slot-scope="scope">
            <el-tag effect="dark" v-if="scope.row.level === '0'">一级</el-tag>
            <el-tag type="success" effect="dark" v-if="scope.row.level === '1'"
              >二级</el-tag
            >
            <el-tag type="warning" effect="dark" v-if="scope.row.level === '2'"
              >三级</el-tag
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rightsList: []
    }
  },
  created() {
    this.getRightlist()
  },
  methods: {
    async getRightlist() {
      const { data: res } = await this.$http.get('rights/list')
      console.log(res)
      if (res.meta.status !== 200) {
        return
      }
      this.rightsList = res.data
    }
  }
}
</script>
<style lang="less" scoped></style>
