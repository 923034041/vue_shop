<template>
   <el-container>
     <!-- 头部区域 -->
  <el-header>
     <div>
        <img src="../assets/logo.png" alt="">
        <span>电商后台管理系统</span>
      </div>
    <el-button type="info" @click="logout">退出</el-button>
  </el-header>
  <!-- 页面主体区域 -->
  <el-container>
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px' :'200px'">
       <div class="toggle-button" @click="toggleCollpse">|||</div>
      <el-menu :collapse="isCollapse" unique-opened background-color="#333744" text-color="#fff"
      active-text-color="#ffd04b"  :collapse-transition="false" router :default-active="activePath">
        <!-- 一级菜单 -->
      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
        <!-- 一级菜单的模板区域 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconsObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
          <el-menu-item :key = "subItem.id"  v-for="subItem in item.children" :index="'/'+subItem.path" @click="saveNavState('/' + subItem.path)">
        <template slot="title">
          <!-- 图标 -->
          <i class="el-icon-location"></i>
          <!-- 文本 -->
          <span>{{subItem.authName}}</span>
        </template>
          </el-menu-item>
      </el-submenu>
    </el-menu>
    </el-aside>
    <!-- 右侧内容 -->
    <el-main>
      <!-- 路由占位符 -->
      <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
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
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error('res.meta.msg')
      this.menulist = res.data
    },
    // 点击按钮，切换菜单的折叠
    toggleCollpse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="scss" scoped>
.el-container{
  height: 100%;
.el-header {
  display: flex;
  justify-content: space-between;
  padding-left: 0px;
  align-items: center;
  font-size: 20px;
  color: #fff;
 >div {
   display: flex;
   align-items: center;
  img {
    width: 50px;
    height: 50px;
    align-items: center;
  }
  span {
    margin-left: 10px;
  }
 }
  background-color: #373d41;
}
 .el-aside {
    background-color: #333744;
    height: 100%;
    .toggle-button {
      background-color: #4A5064;
      text-align: center;
      color: #fff;
      line-height: 24px;
      letter-spacing: 0.2em;
    }
    .el-menu {
       border-right: none;
       .iconfont {
        margin-right: 10px;
      }
    }
   }
 }
</style>
