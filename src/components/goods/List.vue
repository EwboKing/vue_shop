<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">

        <!-- 搜索按钮 -->
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            @clear="getGoodsList"
            clearable>
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>

        <!-- 添加按钮 -->
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table 表格区域 -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="90px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" class="el-icon-edit" size="mini" @click="goEditPage(scope.row.goods_id)"> 编辑</el-button>
            <el-button type="danger" class="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"> 删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background>
      </el-pagination>
    </el-card>

  </div>
</template>

<script>
export default {
  name: 'List',
  data () {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表
      goodsList: [],
      // 总数据个数
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 获取商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品列表失败!')

      // this.$message.success('获取商品列表成功!')
      // console.log(res)
      this.goodsList = res.data.goods
      this.total = res.data.total
    },

    // 监听 pagesize 的改变
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },

    // 监听 pagenum 的改变
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },

    // 根据 id 删除商品
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除商品, 是否继续?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      // 发起删除商品的网络请求
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除商品失败!')
      this.$message.success('删除商品成功!')
      this.getGoodsList()
    },

    // 添加商品的页面
    goAddPage () {
      this.$router.push('/goods/add')
    },

    // 编辑商品的页面
    goEditPage (id) {
      // 发送商品 id 给 edit 组件
      this.$router.push({
        name: 'editpage',
        params: { goodsId: id }
      })
    }
  },
  computed: {

  }
}
</script>

<style lang="less" scoped>
</style>
