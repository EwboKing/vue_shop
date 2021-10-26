<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/logo.png">
        <span>电商后台管理系统</span>
      </div>
      <el-button @click="logout" type="info">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse?'64px':'200px'">

        <!-- 展开所有一级菜单 -->
        <div class="toggle-button" @click="toggleCollapse">
            <i class="iconfont icon-menu"></i>
          </div>

        <!-- 侧边栏菜单区域 -->
        <el-menu
          background-color="#313743"
          text-color="#fff"
          active-text-color="#466eff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >

          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单 -->
            <el-menu-item
              :index="subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNameState(subItem.path)"
            >
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>

          </el-submenu>
        </el-menu>
      </el-aside>

      <!-- 右侧内容主体 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      // 左侧菜单
      menuList: '',
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      // 清除token
      window.sessionStorage.clear()
      // 重定向到登录页面
      this.$router.push('/login')
    },
    // 获取所有的菜单
    async getMenuList () {
      // const { data: res } = await this.$http.get('http://127.0.0.1:8888/api/private/v1/menus')
      const { data: res } = await this.$http.get('/menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      // console.log(this.menuList)
    },

    // 点击按钮实现切换菜单的折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },

    // 保存链接的激活状态
    saveNameState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }

  }
}
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #363d40;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;

    >div {
      display: flex;
      align-items: center;

      span {
        padding-left: 15px;
      }
    }

    img {
      height: 50px;
      border-radius: 50%;
      padding: 3px;
      border: 1px solid #71787b;
    }

  }

  .el-aside {
    background-color: #313743;
    .el-menu{
      border-right: none;
    }
  }

  .el-main {
    background-color: #e9edf1;
  }

  .iconfont{
    padding-right: 10px;
  }

  .toggle-button{
    background-color: #262d3d;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
</style>
