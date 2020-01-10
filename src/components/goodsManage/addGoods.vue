<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片区，用布局容器布局元素 -->
    <el-card>
      <!-- 弹出框 -->
      <el-alert title="添加商品信息" type="info" show-icon center :closable="false"> </el-alert>
      <!-- 进度条 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- form表单 -->
      <el-form ref="addGoodsRef" :model="addGoodsForm" :rules="addGoodsFormRules" label-width="80px" label-position="top">
        <!-- tabs标签页 -->
        <!-- 此处是出发点,切换的时候会将name的值保存到data中的数据中 -->
        <el-tabs :tab-position="tabPosition" v-model="activeIndex" :before-leave="beforeTagLeave" @tab-click="tabClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addGoodsForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addGoodsForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addGoodsForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader :options="allGoodsCategories" :props="cateProps" v-model="addGoodsForm.goods_cat" @change="getThirdCateId"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="item in manyParamsArray" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals_default">
                <el-checkbox :label="item1" v-for="(item1, i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyParamsArray" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :on-success="uploadSuccess" :headers="handleHeaders">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addGoodsForm.goods_introduce"> </quill-editor>
            <el-button type="primary" class="addbtn" @click="addGood">添加商品</el-button>
          </el-tab-pane>
          <el-tab-pane label="完成" name="5">完成</el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="showPreviewDialog">
      <img :src="picAbsolutePath" alt="" width="100%" height="60%" />
    </el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      tabPosition: 'left',
      // 作为数据双向绑定的数据源, 就是表单中填入的时候会发生改变,不依赖后台,用户输入内容时会数据会跟着发生改变......而用来渲染级联选项的数据则来自于后台.
      addGoodsForm: {
        goods_name: '请输入要添加的商品',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addGoodsFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
          { min: 3, message: '长度需要大于3', trigger: 'blur' }
        ],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      allGoodsCategories: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      manyParamsArray: [],
      onlyParamsArray: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      handleHeaders: { Authorization: window.sessionStorage.getItem('token') },
      showPreviewDialog: false,
      picAbsolutePath: ''
    }
  },
  created() {
    this.getGoodsCateList()
  },
  computed: {
    theThirdCateId() {
      if (this.addGoodsForm.goods_cat.length !== 3) {
        return null
      }
      return this.addGoodsForm.goods_cat[this.addGoodsForm.goods_cat.length - 1]
    }
  },
  methods: {
    // 区取所有商品列表.
    async getGoodsCateList() {
      const { data: res } = await this.$http.get('categories')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$remindMes.error('商品分类获取失败')
      }
      this.allGoodsCategories = res.data
      // console.log(this.allGoodsCategories)
    },
    // 如果获取到三级商品分类（选中三级分类触发）的id值， 就发起请求，
    async getThirdCateId() {
      if (this.addGoodsForm.goods_cat.length !== 3) {
        this.addGoodsForm.goods_cat = []
      }
    },
    // 切换标签之前的钩子函数,返回false或者promise就会阻止切换,
    beforeTagLeave(activeName, oldActiveName) {
      // console.log(activeName, oldActiveName)
      if (oldActiveName === '0' && this.addGoodsForm.goods_cat.length !== 3) {
        this.$remindMes.error('请先选择商品分类')
        return false
      }
    },
    // 切换tab时触发的函数
    async tabClicked() {
      // console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.theThirdCateId}/attributes`, {
          params: { sel: 'many' }
        })
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$remindMes.error('商品参数获取失败')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.trim().length === 0 ? [] : item.attr_vals.split(' ')
          item.attr_vals_default = [...item.attr_vals]
        })
        this.manyParamsArray = res.data
        console.log(this.manyParamsArray)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.theThirdCateId}/attributes`, {
          params: { sel: 'only' }
        })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$remindMes.error('属性获取失败')
        }
        this.onlyParamsArray = res.data
        console.log(this.onlyParamsArray)
      }
    },
    // 图片上传后的预览功能
    handlePreview(file) {
      console.log(file)
      const picURL = file.response.data.url
      this.picAbsolutePath = picURL
      this.showPreviewDialog = true
    },
    // 图片上传的移除操作。
    handleRemove(file) {
      console.log(file)
      const temPath = { pic: file.response.data.tmp_path }
      this.addGoodsForm.pics.pop(temPath)
      console.log(this.addGoodsForm.pics)
    },
    // upload组件内部属性，当图片上传成功后自动调用。
    uploadSuccess(response) {
      console.log(response)
      const picTemporaryPath = { pic: response.data.tmp_path }
      this.addGoodsForm.pics.push(picTemporaryPath)
      console.log(this.addGoodsForm.pics)
    },
    // 点击添加商品按钮触发的事件.
    addGood() {
      this.$refs.addGoodsRef.validate(async valid => {
        if (!valid) {
          this.$remindMes.error('请输入符合要求的商品信息')
        }
        const cloneAddForm = _.cloneDeep(this.addGoodsForm)
        cloneAddForm.goods_cat = cloneAddForm.goods_cat.join(',')
        // 处理动态属性
        this.manyParamsArray.forEach(item => {
          const goodsDynamicInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addGoodsForm.attrs.push(goodsDynamicInfo)
        })
        // 处理静态参数
        this.onlyParamsArray.forEach(item => {
          const goodsStaticInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addGoodsForm.attrs.push(goodsStaticInfo)
        })
        cloneAddForm.attrs = this.addGoodsForm.attrs
        const { data: res } = await this.$http.post('goods', cloneAddForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$remindMes.error('商品添加失败')
        }
        this.$router.push('/goods')
        this.$remindMes.success('商品添加成功!')
      })
    }
  }
}
</script>
<style lang="less" scoped>
.el-checkbox {
  margin: 0 7px 0 0;
}
.addbtn {
  margin-top: 60px;
}
.el-steps {
  display: flex;
  justify-content: center;
}
</style>
