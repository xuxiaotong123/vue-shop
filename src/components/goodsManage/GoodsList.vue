<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片区，用布局容器布局元素 -->
    <el-card>
      <el-row :gutter="16">
        <el-col :span="10">
          <div>
            <!-- 搜索框 -->
            <el-input placeholder="请输入内容" class="input-with-select" clearable v-model="queryInfo.query">
              <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="5">
          <div>
            <!-- 添加商品按钮 -->
            <el-button type="primary" @click="goAddGoodsPage">添加商品</el-button>
          </div>
        </el-col>
      </el-row>
      <!-- 商品列表区 -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="expand"></el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column prop="goods_price" label="商品价格" width="110px"> </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="90px"> </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="200px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormate }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <!-- {{ scope.row }} -->
            <el-button type="primary" icon="el-icon-edit" circle @click="editGood(scope.row.goods_id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" circle @click="delCurrentGood(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[20, 30, 50, 80]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改商品对话框 -->
    <el-dialog title="修改商品" :visible.sync="modifyGoodsDialogVisible" width="50%" @close="modifyGoodsDialogClosed">
      <el-form ref="modifyGoodsRef" :model="modifyGoodsForm" :rules="modifyGoodsFormRules" label-width="80px">
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="modifyGoodsForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="modifyGoodsForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number">
          <el-input v-model="modifyGoodsForm.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="modifyGoodsForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyGoodsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyGoodsSure">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0,
      modifyGoodsDialogVisible: false,
      modifyGoodsForm: {},
      modifyGoodsFormRules: {
        goods_name: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, message: '长度需要大于3', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('商品列表获取失败')
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
      // console.log(this.goodsList)
      // this.$remindMes.success('商品列表获取成功')
    },
    // 点击编辑按钮要做的事：1.显示一个编辑的对话框 2.拿到当前行的数据并将其渲染到对话框上 3.对话框关闭（差号/取消/确定{确定时是通过对话框自身绑定的属性的bool值来控制的}）时该什么  表单重置，4.确定时先验证输入是否符合要求 ,如果符合要求 则发送请求,发送最新数据保存到数据库，重新刷新页面，关闭对话框。
    async editGood(id) {
      const { data: res } = await this.$http.get(`goods/${id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('商品查询失败')
      }
      this.modifyGoodsForm = res.data
      console.log(this.modifyGoodsForm)
      // this.$remindMes.success('商品查询成功')
      this.modifyGoodsDialogVisible = true
    },
    // 关闭编辑对话框
    modifyGoodsDialogClosed() {
      // resetField中的f一定要大写 ☆☆☆☆☆☆☆☆☆☆
      this.$refs.modifyGoodsRef.resetFields()
    },
    // 点击编辑对话框上的确定按钮确认提交。
    modifyGoodsSure() {
      // console.log(123)
      this.$refs.modifyGoodsRef.validate(async valid => {
        if (!valid) {
          return this.$remindMes.error('商品名称不符合要求,请重新输入!')
        }
        // 因为对话框已经是脱离出来的二级页面,所以无法通过scope.row的方法拿到当前行的值,只能通过data中的数据拿,而data中的数据是通过发送请求从后台拿到的(查询),   编辑按钮属于当前行,可以通过scope.row的方法拿到当前行的id,scope.row是在当前页面渲染成功的时候就已经存在了.
        const { data: res } = await this.$http.put(`goods/${this.modifyGoodsForm.goods_id}`, {
          goods_name: this.modifyGoodsForm.goods_name,
          goods_price: this.modifyGoodsForm.goods_price,
          goods_number: this.modifyGoodsForm.goods_number,
          goods_weight: this.modifyGoodsForm.goods_weight
        })
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$remindMes.error('编辑商品失败')
        }
        // this.getGoodsList()
        this.$remindMes.success('编辑商品成功')
        this.modifyGoodsDialogVisible = false
      })
    },
    // 删除当前商品
    delCurrentGood(id) {
      this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`goods/${id}`)
          // console.log(res)
          if (res.meta.status !== 200) {
            this.$remindMes({
              type: 'info',
              message: '删除商品失败'
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
    },
    // 手动设置当前页面展示数据的总条数
    handleSizeChange(appointedPageSize) {
      this.queryInfo.pagesize = appointedPageSize
      this.getGoodsList()
    },
    // 手动设置当前页面展示第几页的数据.
    handleCurrentChange(appointedPageNum) {
      this.queryInfo.pagenum = appointedPageNum
      this.getGoodsList()
    },
    // 点击添加商品按钮,显示添加商品对话框.
    // showAddGoodsDialog() {
    //   this.addGoodsDialogVisible = true
    // },
    // // 点击对话框上的确定按钮,确认添加商品
    // addGoodsSure() {
    //   this.$refs.addGoodsRef.validate(valid => {
    //     if (!valid) {
    //       return this.$remindMes.error('请输入符合要求的商品名称!')
    //     }
    //     this.$http.post('goods')
    //     this.$remindMes.success('添加商品成功!')
    //   })
    //   this.addGoodsDialogVisible = false
    // },
    // // 关闭添加商品对话框
    // addGoodsDialogClosed() {
    //   this.$refs.addGoodsRef.resetFields()
    // }
    goAddGoodsPage() {
      this.$router.push('/goods/add')
    }
  }
}
</script>
<style lang="less" scoped></style>
