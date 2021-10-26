<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>

      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">

        <!-- 搜索按钮 -->
        <el-col :span="12">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>

        <!-- 添加按钮 -->
        <el-col :span="7">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 用户信息展示区域 -->
      <el-table :data="userList" border stripe>
        <el-table-column label="序号" type="index"></el-table-column>
        <el-table-column label="用户名" prop="username"></el-table-column>
        <el-table-column label="E-mail" prop="email"></el-table-column>
        <el-table-column label="Mobile" prop="mobile"></el-table-column>
        <el-table-column label="权限" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#9c9c9c"
              @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope" width="180px">
            <!-- 修改按钮 -->
            <el-tooltip effect="dark" content="编辑" placement="top" :enterable="false">
              <el-button type="primary" class="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">
              </el-button>
            </el-tooltip>

            <!-- 删除图标 -->
            <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
              <el-button type="danger" class="el-icon-delete" size="mini" @click="removedUserById(scope.row.id)"></el-button>
            </el-tooltip>

            <!-- 管理权限按钮 -->
            <el-tooltip effect="dark" content="设置" placement="top" :enterable="false">
              <el-button type="warning" class="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>

    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">

      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <!-- 用户名 -->
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <!-- 邮箱 -->
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <!-- 手机号码 -->
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog title="修改用户信息" :visible.sync="EditDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <!-- 用户名 -->
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>

        <!-- 邮箱 -->
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>

        <!-- 手机号码 -->
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="EditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Users',
  data () {
    // 验证邮箱的自定义规则
    var checkEmail = (rule, value, callback) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) return callback()

      callback(new Error('请输入合法的邮箱'))
    }

    // 验证手机号码的自定义规则
    var checkMobile = (rule, value, callback) => {
      // 验证手机号码的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) return callback()

      callback(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取用户列表参数
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前页面显示的数据个数
        pagesize: 2
      },
      userList: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户表单的验证规则
      addFormRules: {
        username: [{
          required: true,
          message: '请输入用户名',
          trigger: 'blur'
        },
        {
          min: 3,
          max: 10,
          message: '用户名的长度必须在3-10个字符之间',
          trigger: 'blur'
        }
        ],
        password: [{
          required: true,
          message: '请输入密码',
          trigger: 'blur'
        },
        {
          min: 6,
          max: 15,
          message: '密码的长度必须在6-15个字符之间',
          trigger: 'blur'
        }
        ],
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号码',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      },

      // 控制修改用户对话框的显示与隐藏
      EditDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      // 修改用户表单的验证规则
      editFormRules: {
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号码',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      }
    }
  },
  created () {
    this.getUserList()
    // console.log(this)
  },
  methods: {
    // 获取用户列表的事件
    getUserList () {
      this.$http.get('users', {
        params: this.queryInfo
      }).then(
        response => {
          if (response.data.meta.status !== 200) return this.$message.error('获取用户列表失败')
          this.userList = response.data.data.users
          this.total = response.data.data.total
          // console.log(response.data)
          // console.log(this.userList)
        },
        error => {
          console.log(error.message)
        }
      )
    },

    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      // console.log(newSize)
      this.getUserList()
    },

    // 监听页码值改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      // console.log(newPage)
      this.getUserList()
    },

    // 监听switch开关状态的改变
    userStateChanged (userinfo) {
      // console.log(userinfo)
      this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`).then(
        response => {
          if (response.data.meta.status !== 200) {
            userinfo.mg_state = !userinfo.mg_state
            return this.$message.error('更新用户状态失败!!!')
          }
          this.$message.success('更新用户状态成功!')
        },
        error => {
          console.log(error.message)
        }
      )
    },

    // 监听关闭添加用户对话框的事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },

    // 监听关闭修改用户对话框的事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },

    // 添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return this.$message.error('用户信息校验失败')
        // 可以发起添加用户的网络请求
        const {
          data: res
        } = await this.$http.post('/users', this.addForm)
        // console.log(res)
        if (res.meta.status !== 201) this.$message.error('添加用户失败')
        this.$message.success('添加用户成功')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 刷新用户列表
        this.getUserList()
      })
    },

    // 展示编辑用户的对话框
    async showEditDialog (id) {
      // console.log(id)
      const {
        data: res
      } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询用户信息失败')
      this.editForm = res.data
      this.EditDialogVisible = true
    },

    // 修改用户信息并预验证
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 发起用户信息的数据请求
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 200) return this.$message.error('修改用户信息失败!!!')

        // 关闭修改用户对话框
        this.EditDialogVisible = false
        // 刷新数据列表
        this.getUserList()
        // 提示数据修改成功
        this.$message.success('修改用户信息成功!')
      })
    },

    // 根据id删除用户的信息
    async removedUserById (id) {
      // console.log(id)
      const { data: tipRes } = await this.$http.get('users/' + id)
      // 弹框询问用户是否删除
      // console.log(res)
      const confirmResult = await this.$confirm(`此操作将永久删除用户${tipRes.data.username}, 是否继续?`, '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      // 如果用户确认删除,则返回值为字符串 confirm
      // 如果用户取消删除,则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') return this.$message.info('已取消用户删除行为')

      // 发起删除用户的网络请求
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户信息失败')
      this.$message.success('删除用户信息成功')
      this.getUserList()
    }

    //
  }
}
</script>

<style lang="less" scoped>
</style>
