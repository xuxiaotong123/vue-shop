<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数" show-icon type="warning" effect="dark" closable> </el-alert>
      <el-row :gutter="16">
        <el-col :span="10">选择商品分类 :</el-col>
        <el-col :span="10"><el-cascader v-model="selectedCateIdsArray" :options="allCateList" @change="handleChange" :props="appointedProps" change-on-select></el-cascader></el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加动态参数 -->
          <el-button type="primary" :disabled="isAddBtndisabled" @click="showAddParamsDialog">添加参数</el-button>
          <!-- 添加动态参数表格 -->
          <el-table :data="manyListArray" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- v-for循环  :key的值要与被循环的对象有关联☆☆☆☆☆ -->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="delDynamicParamsChild(i, scope.row)">
                  {{ item }}
                </el-tag>
                <!-- <pre>{{ scope.row }}</pre> -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- <pre>{{ scope.row }}</pre> -->
                <el-button type="primary" icon="el-icon-edit" @click="showModifyParamsDialog(scope.row)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="showDelParamsConfirm(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isAddBtndisabled" @click="showAddParamsDialog">添加属性</el-button>
          <!-- 添加静态参数表格 -->
          <el-table :data="onlyListArray" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- v-for循环  :key的值要与被循环的对象有关联☆☆☆☆☆ -->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="delDynamicParamsChild(i, scope.row)">
                  {{ item }}
                </el-tag>
                <pre>{{ scope.row }}</pre>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" @click="showModifyParamsDialog(scope.row)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="showDelParamsConfirm(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加动态和静态参数的共公对话框 -->
    <el-dialog :title="'添加' + isAddDynamicOrstatic" :visible.sync="addParamsDialogVisible" width="50%" @close="addParamsDialogClosed">
      <!-- 添加参数对话框的form表单 -->
      <el-form ref="addParamsRef" :model="addParamsForm" :rules="addFormRules" label-width="80px">
        <el-form-item :label="isAddDynamicOrstatic" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParamsSure">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改动态和静态参数的共公对话框 -->
    <el-dialog :title="'修改' + isAddDynamicOrstatic" :visible.sync="modifyParamsDialogVisible" width="50%" @close="modifyParamsDialogClosed">
      <!-- 修改参数对话框的form表单 -->
      <el-form ref="modifyParamsRef" :model="modifyParamsForm" :rules="modifyFormRules" label-width="80px">
        <el-form-item :label="isAddDynamicOrstatic" prop="attr_name">
          <el-input v-model="modifyParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyParamsSure">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      allCateList: [],
      selectedCateIdsArray: [],
      appointedProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      onlyListArray: [],
      manyListArray: [],
      addParamsDialogVisible: false,
      addParamsForm: {},
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ]
      },
      modifyParamsDialogVisible: false,
      modifyParamsForm: {},
      modifyFormRules: {
        attr_name: [
          { required: true, message: '请输入新的参数名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    isAddBtndisabled() {
      if (this.selectedCateIdsArray.length !== 3) {
        return true
      }
      return false
    },
    theThirdCateId() {
      if (this.selectedCateIdsArray.length !== 3) {
        return false
      }
      return this.selectedCateIdsArray[2]
    },
    isAddDynamicOrstatic() {
      if (this.activeName === 'many') {
        return '动态属性'
      }
      return '静态参数'
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('商品列表获取失败')
      }
      this.allCateList = res.data
      // this.$remindMes.success('商品列表获取成功')
    },
    // 级联表选择的时候自动触发的函数。
    handleChange() {
      this.getSelectedThirdCateParams()
    },
    // tab点击切换tab栏时触发的事件
    handleTabClick() {
      this.getSelectedThirdCateParams()
    },
    // 获取选中的三级分类的参数对应的属性数组。
    async getSelectedThirdCateParams() {
      if (this.selectedCateIdsArray.length !== 3) {
        this.selectedCateIdsArray = []
        // this.onlyListArray = []
        // this.manyListArray = []
        return
      }
      // console.log(this.selectedCateIdsArray)
      // console.log(this.theThirdCateId)
      // console.log(this.theThirdCateId)
      const { data: res } = await this.$http.get(`categories/${this.theThirdCateId}/attributes`, {
        params: { sel: this.activeName }
      })
      console.log(res)
      if (res.meta.status !== 200) {
        this.$remindMes.error('分类参数获取失败')
      }
      res.data.forEach(item => {
        // 判断一个变量是否为空
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'only') {
        this.onlyListArray = res.data
        console.log(this.onlyListArray)
      } else if (this.activeName === 'many') {
        this.manyListArray = res.data
        console.log(this.manyListArray)
      }
      // this.$remindMes.success('分类参数获取成功')
    },
    // 添加参数对话框关闭时，重置表单。
    addParamsDialogClosed() {
      this.$refs.addParamsRef.resetFields()
    },
    showAddParamsDialog() {
      this.addParamsDialogVisible = true
    },
    // 点击对话框的确定按钮触发的函数。
    addParamsSure() {
      this.$refs.addParamsRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.theThirdCateId}/attributes`, { attr_name: this.addParamsForm.attr_name, attr_sel: this.activeName })
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$remindMes.error('添加参数失败')
        }
        // this.addParamsForm = {}
        // 请求成功后会关闭对话框，会触发对话框自带的close事件，而此事件中会重置表单。
        this.$remindMes.success('添加参数成功')
        this.getSelectedThirdCateParams()
        this.addParamsDialogVisible = false
      })
    },
    // 关闭对话框时重置表单。
    modifyParamsDialogClosed() {
      this.$refs.modifyParamsRef.resetFields()
    },
    // 点击修改按钮弹出修改按钮圹框
    async showModifyParamsDialog(row) {
      // this.attr_id = row.attr_id
      // this.attr_name = row.attr_name
      const { data: res } = await this.$http.get(`categories/${this.theThirdCateId}/attributes/${row.attr_id}`, { attr_sel: this.activeName })
      if (res.meta.status !== 200) {
        return this.$remindMes.error('查询参数失败')
      }
      this.$remindMes.success('查询参数成功')
      this.modifyParamsForm = res.data
      console.log(this.modifyParamsForm)
      this.modifyParamsDialogVisible = true
    },
    // 点击修改对话框确定按钮触发的函数。
    modifyParamsSure() {
      this.$refs.modifyParamsRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.theThirdCateId}/attributes/${this.modifyParamsForm.attr_id}`, {
          attr_name: this.modifyParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$remindMes.error(res.meta.status)
        }
        console.log(res)
        this.getSelectedThirdCateParams()
        this.$remindMes.success('参数更新成功')
      })
      this.modifyParamsDialogVisible = false
    },
    // 点击删除按钮删除参数。
    showDelParamsConfirm(attrId) {
      this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`categories/${this.theThirdCateId}/attributes/${attrId}`)
          console.log(res)
          if (res.meta.status !== 200) {
            this.$remindMes({
              type: 'info',
              message: '删除参数失败'
            })
            return
          }
          this.getSelectedThirdCateParams()
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
    showInput(currentRow) {
      currentRow.inputVisible = true
      // nextT相当于延时效果，等到页面加载完成时才去获得焦点。
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // newtag标签失去焦点或者按下enter键时，发送请求添加参数
    async handleInputConfirm(currentRow) {
      if (currentRow.inputValue.trim().length === 0) {
        currentRow.inputValue = ''
        currentRow.inputVisible = false
        return
      }
      // 此处的执行顺序？？？？
      currentRow.attr_vals.push(currentRow.inputValue)
      currentRow.inputValue = ''
      currentRow.inputVisible = false
      const { data: res } = await this.$http.put(`categories/${this.theThirdCateId}/attributes/${currentRow.attr_id}`, {
        attr_name: currentRow.attr_name,
        attr_sel: currentRow.attr_sel,
        attr_vals: currentRow.attr_vals.join(' ')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        currentRow.attr_vals.splice(currentRow.attr_vals[currentRow.attr_vals.length - 1])
        return this.$remindMes.error('添加参数失败')
      }
      // 页面的标签是通过对数组的操作来渲染的，不发送请求也可以渲染，并不依赖后台请求的数据，但是后端数据要与页面数据保持一致，所以当有动作触发时，先发请求让后台保存最新数据再渲染。
      this.$remindMes.success('添加参数成功')
    },
    // 删除动态参数下子级参数
    async delDynamicParamsChild(index, currentRow) {
      currentRow.attr_vals.splice(index, 1)
      const { data: res } = await this.$http.put(`categories/${this.theThirdCateId}/attributes/${currentRow.attr_id}`, {
        attr_name: currentRow.attr_name,
        attr_sel: currentRow.attr_sel,
        attr_vals: currentRow.attr_vals.join(' ')
      })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('删除参数失败')
      }
      // 页面的标签是通过对数组的操作来渲染的，不发送请求也可以渲染，并不依赖后台请求的数据，但是后端数据要与页面数据保持一致，所以当有动作触发时，先发请求让后台保存最新数据再渲染。
      // 粘贴需要谨慎☆☆☆☆☆☆☆☆☆☆☆☆☆
      // currentRow.attr_vals.push(currentRow.inputValue)
      // currentRow.inputValue = ''
      // currentRow.inputVisible = false
      this.$remindMes.success('删除参数成功')
    }
  }
}
</script>
<style scoped>
.el-row {
  margin-top: 15px;
}
.el-table {
  margin-top: 15px;
}
.input-new-tag {
  width: 120px;
}
</style>
