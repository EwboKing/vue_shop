<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-button type="primary" size="mini" class="cancelBtn" @click="cancelEdit">取消编辑</el-button>
      <!-- 提示区域 -->
      <el-alert
        title="编辑商品信息"
        type="info"
        center
        show-icon
        :closable="false">
      </el-alert>

      <!-- 步骤条 -->
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- tab 栏区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top">
        <el-tabs
          v-model="activeIndex"
          :tab-position="'left'"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked">
          <!-- 基本信息 -->
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="editForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="editForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <!-- 商品的级联选择框 -->
              <el-cascader
                v-model="editForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChanged"
                clearable>
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品参数 -->

          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的 item 项 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>

          <!-- 商品属性 -->
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品图片 -->
          <el-tab-pane label="商品图片" name="3">
            <!--
              action 图片要上传的后台 API 地址
             -->
            <el-upload
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <!-- 商品内容 -->
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器 -->
            <quill-editor
              v-model="editForm.goods_introduce">
            </quill-editor>
            <!-- 添加商品的按钮 -->
            <el-button type="primary" class="editBtn" @click="editGoods">确认修改</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 预览图片的对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%">
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'Add',
  data () {
    return {
      activeIndex: '0',
      // 编辑商品的 id
      // goodsId: '',
      // 编辑商品的数据表单对象
      editForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属的分类数组
        goods_cat: [],
        // 图片
        pics: [],
        // 商品的详情描述
        goods_introduce: '',
        attrs: []
      },
      // 添加商品的数据表单验证规则
      editFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      // 商品分类数据
      cateList: [],
      // 级联选择器的配置对象
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 动态参数列表数据
      manyTableData: [],
      // 静态属性列表数据
      onlyTableData: [],
      // 上传图片的 URL 地址
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的 headers 请求头对象
      headerObj: { Authorization: window.sessionStorage.getItem('token') },
      previewPath: '',
      // 预览图片对话框的显示与隐藏
      previewVisible: false
    }
  },
  created () {
    this.getCateList()
    // this.getCateById(this.$route.params.goodsId)
    this.getGoodsForm(this.$route.params.goodsId)
    // window.sessionStorage.setItem('editPageCateId', this.$route.params.goodsId)
  },
  beforeMount () {
    // console.log(this.$route.params)
  },
  methods: {
    // 获取当前商品的数据表单
    async getGoodsForm (goodsId) {
      const { data: res } = await this.$http.get(`goods/${goodsId}`)
      // console.log(res)
      // console.log(window.sessionStorage.getItem('editPageCateId'))
      if (res.meta.status !== 200) {
        // console.log(goodsId)
        this.$router.push('/goods')
        return this.$message.error('获取商品数据失败,请重新选择要编辑的商品!')
      }

      this.editForm = res.data
      // console.log(this.editForm)
      this.editForm.goods_cat = this.editForm.goods_cat.split(',').map(Number)
      // console.log(this.editForm.goods_cat, '$')
    },

    // 获取当前商品分类
    async getCateById (id) {
      const { data: res } = await this.$http.get(`categories/${id}`)
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败!')
    },

    // 获取所有商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败!')

      this.cateList = res.data
      // console.log(this.cateList)
    },

    // 级联选择器选中项变化触发此函数
    handleChanged () {
      // console.log(this.editForm.goods_cat)
      if (this.editForm.goods_cat.length !== 3) this.editForm.goods_cat = []
    },

    // 监听离开标签页的放行
    beforeTabLeave (activeName, oldActiveName) {
      // console.log(activeName, oldActiveName)
      if (oldActiveName === '0' && this.editForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类!')
        return false
      }
    },

    // 表单点击触发事件
    async tabClicked () {
      // console.log(typeof (this.activeIndex))
      // 访问的是动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) return this.$message.error('获取动态参数列表失败!')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        // console.log(res.data)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) return this.$message.error('获取静态属性列表失败!')
        // console.log(res.data)
        this.onlyTableData = res.data
      }
    },

    // 处理图片预览效果
    handlePreview (file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },

    // 处理移除图片的操作
    handleRemove (file) {
      // console.log(file)
      // 1.获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2.从 pics 数组中,找到这个图片对应的索引值
      const i = this.editForm.pics.findIndex(x => x.pic === filePath)
      // 3.调用数组的 splice 方法,把图片信息对象从 pics 数组中移除
      this.editForm.pics.splice(i, 1)
      // console.log(this.editForm)
    },

    // 监听图片上传成功的事件
    handleSuccess (response) {
      // console.log(response)
      // 1.拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2.将图片信息对象 push 到 pics 数组中
      this.editForm.pics.push(picInfo)
      // console.log(this.editForm)
    },

    // 修改商品
    editGoods () {
      // console.log(this.$router)
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项')

        // 执行修改的业务逻辑
        const form = _.cloneDeep(this.editForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
          this.editForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.editForm.attrs.push(newInfo)
        })
        form.attrs = this.editForm.attrs
        // console.log(form)

        // 发起请求 商品的名称是唯一的
        const { data: res } = await this.$http.put(`goods/${this.$route.params.goodsId}`, form)
        if (res.meta.status !== 200) {
          // console.log(res)
          return this.$message.error('修改商品失败!')
        }
        this.$message.success('修改商品成功!')
        this.$router.push('/goods')
      })
    },

    // 取消修改商品信息
    cancelEdit () {
      this.$router.push('/goods')
    }
  },
  computed: {
    cateId () {
      if (this.editForm.goods_cat.length === 3) {
        return this.editForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
  .el-checkbox{
    margin: 5px 10px 0 0 !important;
  }

  .previewImg{
    width: 100%;
  }

  .editBtn{
    margin-top: 15px;
  }

  .cancelBtn{
    margin-bottom: 15px;
  }
</style>
