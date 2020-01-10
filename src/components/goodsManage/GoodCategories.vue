<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 添加分类 -->
      <el-row>
        <el-col :span="10">
          <div>
            <!-- 添加分类按钮 -->
            <el-button type="primary" @click="showAddGoodsDialog">添加分类</el-button>
          </div>
        </el-col>
      </el-row>
      <!-- table表格区 -->
      <tree-table :data="goodsCatesList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
        <template slot="isok" slot-scope="scope">
          <!-- <pre>{{scope.row}}</pre> -->
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:green; font-size:20px"></i>
          <i class="el-icon-error" v-if="scope.row.cat_deleted === true"></i>
        </template>
        <template slot="sort" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag v-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag v-if="scope.row.cat_level === 2">三级</el-tag>
        </template>
        <template slot="operation" slot-scope="scope">
          <!-- <pre>{{ scope.row }}</pre> -->
          <el-button type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" @click="delCate(scope.row.cat_id)">删除当前</el-button>
        </template>
      </tree-table>
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
      <!-- 添加商品对话框 -->
      <el-dialog title="添加分类" :visible.sync="addGoodCateDialogVisible" width="50%" @close="clearParentCate">
        <el-form ref="addCateRef" :model="addCateForm" :rules="addCateFormRules" label-width="100px">
          <el-form-item label="分类名称:" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类:">
            <el-cascader v-model="selectedCateIdsArray" :options="parentCateList" @change="handleChange" :props="appointedProps" clearable change-on-select></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addGoodCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addGoodCate">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 定义请求参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 初始商品列表
      goodsCatesList: [],
      total: 0,
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '是否有效', type: 'template', template: 'isok' },
        { label: '排序', type: 'template', template: 'sort' },
        { label: '操作', type: 'template', template: 'operation' }
      ],
      addGoodCateDialogVisible: false,
      // 用来发送请求的数据，需要根据接口文档自己定义  ☆☆☆☆☆☆
      addCateForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
          { min: 4, max: 20, message: '长度在 4 到 20 个字符', trigger: 'blur' }
        ]
      },
      selectedCateIdsArray: [],
      parentCateList: [],
      appointedProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('商品列表获取失败')
      }
      this.goodsCatesList = res.data.result
      this.total = res.data.total
    },
    // 更改当前页面显示数据的条数
    handleSizeChange(newPageSize) {
      // console.log(newPageSize)
      this.queryInfo.pagesize = newPageSize
      this.getGoodsList()
    },
    // 更改当前页为第几页
    handleCurrentChange(newPageSize) {
      // console.log(newPageSize)
      this.queryInfo.pagenum = newPageSize
      this.getGoodsList()
    },
    showAddGoodsDialog() {
      this.getParentCateList()
      this.addGoodCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('前两层分类列表获取失败')
      }
      this.parentCateList = res.data
    },
    // 选中级联中的选项时触发的函数。
    handleChange() {
      if (this.selectedCateIdsArray.length > 0) {
        this.addCateForm.cat_pid = this.selectedCateIdsArray[this.selectedCateIdsArray.length - 1]
        this.addCateForm.cat_level = this.selectedCateIdsArray.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.selectedCateIdsArray)
    },
    // 关闭整个对话框的时候清空级联菜单中的的选中项
    clearParentCate() {
      this.$refs.addCateRef.resetFields()
      // this.addCateForm = {}
      this.selectedCateIdsArray = []
    },
    // 点击对话框中的确定按钮时触发的函数。
    addGoodCate() {
      this.$refs.addCateRef.validate(async valid => {
        if (!valid) {
          this.$refs.addCateRef.resetFields()
          return this.$remindMes.error('请输入符合要求的分类')
        }
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        console.log(res)
        if (res.meta.status !== 201) {
          this.$remindMes.error('添加分类请求失败')
        }
        this.getGoodsList()
        console.log(this.addCateForm)
        this.$remindMes.success('添加分类请求成功')
        this.addGoodCateDialogVisible = false
      })
    },
    // 删除分类
    delCate(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`categories/${id}`)
          console.log(res)
          if (res.meta.status !== 200) {
            this.$remindMes({
              type: 'info',
              message: '删除分类失败'
            })
            return
          }
          this.getGoodsList()
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
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
