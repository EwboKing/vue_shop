<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        title="注意: 只允许为第三级分类设置参数"
        type="warning"
        :closable="false"
        show-icon>
      </el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <!-- 商品的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChanged"
            clearable>
          </el-cascader>
        </el-col>
      </el-row>

      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">

        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialog">添加参数</el-button>

          <!-- 动态参数表格 -->
          <el-table :data="manyTabledata" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染 tag 标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  @close="handleClose(i,scope.row)"
                  closable>{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.addParamsInputVisible"
                  v-model="scope.row.addParamsInputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParamsById(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialog">添加属性</el-button>

          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染 tag 标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  @close="handleClose(i,scope.row)"
                  closable>{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.addParamsInputVisible"
                  v-model="scope.row.addParamsInputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>

            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParamsById(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px">
        <el-form-item :label="contenText + '名称'" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数的对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <!-- 修改参数的对话框 -->
      <el-form
        :model="editForm"
        :rules="addFormRules"
        ref="editFormRef"
        label-width="100px">
        <el-form-item :label="contenText + '名称'" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  data () {
    return {
      // 商品分类列表
      cateList: [],
      // 级联选择框的配置对象
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框绑定的数组
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数的数据
      manyTabledata: [],
      // 静态参数的数据
      onlyTableData: [],
      // 控制添加动态参数/静态属性对话框的显示与隐藏
      addDialogVisible: false,
      // 控制修改动态参数/静态属性对话框的显示与隐藏
      editDialogVisible: false,
      // 添加参数的数据表单对象
      addForm: {
        attr_name: ''
      },
      // 修改参数的数据表单对象
      editForm: {
        attr_name: '',
        attr_id: ''
      },
      // 添加表单数据的验证规则
      addFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      }
    }
  },
  mounted () {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败!')
      this.cateList = res.data
    },

    // 级联选择框变化的回调函数
    handleChanged () {
      this.getParamsData()
    },

    // 获取参数的列表数据
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTabledata = []
        this.onlyTableData = []
        return
      }

      // 选中的为三级分类
      // console.log(this.selectedCateKeys)
      // 根据所选分类的 id 和所处面板获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数列表失败!')
      // console.log(res.data)

      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals = item.attr_vals.split(' ') : [],
        // 控制文本框显示与隐藏
        item.addParamsInputVisible = false
        // 文本框输入的内容
        item.addParamsInputValue = ''
      })

      // console.log(res.data)

      if (this.activeName === 'many') {
        this.manyTabledata = res.data
      } else {
        this.onlyTableData = res.data
      }
    },

    // tab 页签点击事件的处理函数
    handleTabClick () {
      // console.log(this.activeName)
      this.getParamsData()
    },

    // 添加对话框的显示与隐藏
    showAddDialog () {
      this.addDialogVisible = true
    },

    // 修改对话框的显示与隐藏
    showEditDialog (attr) {
      this.editDialogVisible = true
      this.editForm.attr_name = attr.attr_name
      this.editForm.attr_id = attr.attr_id
      // console.log(attr)
    },

    // 监听对话框关闭的处理事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },

    // 监听对话框关闭的处理事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },

    // 添加参数/属性
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('校验数据失败!')
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 201) return this.$message.error('添加失败!')
        this.$message.success('添加成功!')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },

    // 修改参数/属性
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$message.error('校验数据失败!')
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 200) return this.$message.error('更新失败!')
        this.$message.success('更新成功!')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },

    // 删除参数/属性
    async removeParamsById (attrId) {
      const confirmResult = await this.$confirm(`此操作将永久删除此${this.contenText}, 是否继续?`, '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')

      // 发起删除的网络请求
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`)
      if (res.meta.status !== 200) return this.$message.error(`删除${this.contenText}失败!`)
      this.$message.success(`已删除该${this.contenText}`)
      this.getParamsData()
    },

    // 文本框失去焦点或摁下 enter 触发的事件
    async handleInputConfirm (row) {
      if (row.addParamsInputValue.trim().length === 0) {
        row.addParamsInputValue = ''
        row.addParamsInputVisible = false
        return
      }

      // 没有 return 则进行后续处理
      row.attr_vals.push(row.addParamsInputValue.trim())
      row.addParamsInputValue = ''
      row.addParamsInputVisible = false
      // 向服务器发起添加参数的网络请求
      this.saveAttrVals(row)
    },

    // 将对 attr_vals 的操作保存到数据库
    async saveAttrVals (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, { attr_name: row.attr_name, attr_sel: row.attr_sel, attr_vals: row.attr_vals.join(' ') })
      if (res.meta.status !== 200) return this.$message.error('添加失败!')
      this.$message.success('更新成功!')
    },

    // 点击切换输入框和文本框
    showInput (row) {
      row.addParamsInputVisible = true
      // 让文本框自动获得焦点
      // $nextTick 方法的作用,就是当页面上元素被重新渲染之后,才会执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },

    // 删除对应的参数
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }

  },
  computed: {
    // 控制按钮的禁用
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) return true
      return false
    },

    // 当前选择的分类 id
    cateId () {
      if (this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
      return null
    },

    // 动态计算标题的文本
    titleText () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    },

    // 动态计算参数/属性
    contenText () {
      if (this.activeName === 'many') return '参数'
      return '属性'
    }
  }

}
</script>

<style lang="less" scoped>
  .cat_opt{
    margin: 20px 0;
  }

  .el-tag{
    margin: 10px;
  }

  .input-new-tag{
    width: 120px;
  }
</style>
