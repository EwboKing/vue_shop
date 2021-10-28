<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>

      <!-- 添加分类按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 商品分类区域 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-row-hover="false"
        show-index
        stripe
        border
      >
        <!-- 是否有效 -->
        <template slot="isValid" slot-scope="scope">
          <i class="el-icon-success" style="color: #00aa00;" v-if="scope.row.cat_deleted === false"></i>
          <i class="el-icon-error" style="color: red;" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level == 0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level == 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="option" slot-scope="scope">
          <el-button type="primary" class="el-icon-edit" size="mini" @click="showEditCateDialog(scope.row,scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" class="el-icon-delete" size="mini" @click="removedCateById(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>

    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed">
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px">
        <el-form-item label="分类名称 : " prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类 : ">
          <!--
            options 指定数据源
            props 指定配置对象
           -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类的对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClosed">
      <el-form
        :model="editCateForm"
        :rules="addCateFormRules"
        ref="editCateFormRef"
        label-width="100px">
        <el-form-item label="分类名称 : " prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Cate',
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类数据列表
      catelist: [],
      // 总数据条数
      total: 0,
      // 为 tabel 指定列的定义
      columns: [
        {
          label: '分类名称',
          width: '280px',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          width: '150px',
          // 将当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'isValid'
        },
        {
          label: '排序',
          width: '150px',
          // 将当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 将当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'option'
        }
      ],
      // 控制添加分类对话框的隐藏与显示
      addCateDialogVisible: false,
      // 控制编辑分类对话框的隐藏与显示
      editCateDialogVisible: false,
      // 添加分类表单的数据
      addCateForm: {
        // 添加分类的名称
        cat_name: '',
        // 父级分类的 id
        cat_pid: 0,
        // 分类的层级
        cat_level: 0
      },
      // 编辑分类表单的数据
      editCateForm: {
        cat_name: '',
        cate_id: ''
      },
      // 添加分类表单的验证规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选中的父级分类的 id 数组
      selectedKeys: []

    }
  },
  mounted () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      // console.log(res.data)
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败!')
      // 把获得的数据保存到 catelist
      this.catelist = res.data.result
      // 把获得的数据总条数保存到 total
      this.total = res.data.total
      // console.log(this.catelist)
    },

    // 监听 pagesize 改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },

    // 监听 pagenum 改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },

    // 展示添加分类的对话框
    showAddCateDialog () {
      // 获取父级分类的数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },

    // 展示编辑商品分类的对话框
    showEditCateDialog (cateList, id) {
      // console.log(id)
      this.editCateForm.cat_name = cateList.cat_name
      this.editCateForm.cate_id = id
      this.editCateDialogVisible = true
    },
    // 根据 id 删除商品分类
    async removedCateById (id) {
      // console.log(id)
      const confirmResult = await this.$confirm('此操作将永久删除此分类, 是否继续?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') return this.$message.info('已取消删除行为')

      // 发起删除分类的网络请求
      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除分类失败!')
      this.$message.success('已删除该分类')
      this.getCateList()
    },

    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类数据失败!')

      this.parentCateList = res.data
    },

    // 选中项发生变化触发
    parentCateChanged () {
      // console.log(this.selectedKeys)
      // 如果 selectedKeys 的 length 大于 0 ,证明选中了父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的 id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 当前分类的等级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的 id
        this.addCateForm.cat_pid = 0
        // 当前分类的等级
        this.addCateForm.cat_level = 0
      }
    },

    // 点击按钮添加新的分类
    addCate () {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return this.$message.error('校验数据失败!')
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败!')
        this.$message.success('添加分类成功!')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },

    // 点击按钮发送网络请求编辑分类
    async editCate () {
      const { data: res } = await this.$http.put(`categories/${this.editCateForm.cate_id}`, { cat_name: this.editCateForm.cat_name })
      if (res.meta.status !== 200) return this.$message.error('更新分类失败!')
      this.$message.success('更新分类成功!')
      this.getCateList()
      this.editCateDialogVisible = false
    },

    // 监听添加对话框的关闭时间,重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },

    // 监听编辑对话框的关闭时间,重置表单数据
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
  .treeTable{
    margin-top: 20px;
  }

  .el-cascader{
    width: 100%;
  }
</style>
