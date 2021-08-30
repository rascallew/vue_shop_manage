<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
  <el-header>
    <div>
      <div class="icon1">⚡</div>
      <span>电商管理系统</span>
    </div>
    <el-button type="warning" @click="logout">退出</el-button></el-header>
  <!-- 页面主体区域 -->
  <el-container>
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse?'60px':'200px'">
      <div class="toggle-button" @click="toggleCollapse">|||</div>
       <el-menu
      background-color="rgb(172, 216, 183)"
      text-color="rgb(57, 163, 8)"
      active-text-color="#409eff" unique-opened :collapse="isCollapse"
      :collapse-transition="false" :router="true" :default-active="activePath">
      <!-- 一级菜单 -->
      <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
        <!-- 一级菜单模板区域 -->
        <template v-slot:title>
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" 
        :key="subItem.id" @click="saveNavState('/'+subItem.path)">
          <template v-slot:title>
          <i class="el-icon-menu"></i>
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
import axios from 'axios'
export default {
  data() {
    return {
      //左侧菜单数据
      menuList:[],
      iconsObj:{
        147: 'el-icon-magic-stick',
        '125':'el-icon-s-custom',
        '103':'el-icon-s-platform',
        '101':'el-icon-shopping-bag-2',
        '102':'el-icon-wallet',
        '145':'el-icon-s-data'
      },
      //是否折叠 默认不折叠
      isCollapse: false,
      //被激活的链接地址
      activePath:'',
      welcome: {
        authName: '欢迎光临😊',
        id: 147,
        order: 1,
        path: '/welcome',
        children: [
          {
            authName: 'Welcome',
            id: 124,
            order: 1,
            path: 'welcome',
            children: []
          }
        ]
      }

    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
methods:{
  logout() {
    window.sessionStorage.clear()
    this.$router.push('/login')
  },
  //获取所有菜单
  async getMenuList(){
    await axios.get('https://lianghj.top:8888/api/private/v1/menus').then(res => {
      // console.log(res.data);
      if(res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
      res.data.data.unshift(this.welcome)
      this.menuList = res.data.data
    })
  },
  //点击按钮折叠
  toggleCollapse() {
    this.isCollapse = !this.isCollapse
  },
  saveNavState(activePath) {
    window.sessionStorage.setItem('activePath',activePath)
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
    background-color: rgb(172, 216, 183);
    display: flex;
    // 左右贴边对齐
    justify-content: space-between;
    align-items: center;
    span {
      color: rgb(57, 163, 8);
      font-weight: 800;
      font-size: 18px;
      padding-left: 5px;
    }
  }
  .el-aside {
    background-color: rgb(172, 216, 183);
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: rgb(220, 247, 227);
  }
  .icon1 {
    width: 52px;
    height: 52px;
    border-radius: 50%;
    box-shadow: 0 0 10px #ff0;
    background-color: #fff;
    display: inline-block;
    text-align: center;
    line-height: 52px;
    font-size: 25px;
  }
  i {
    padding-right: 5px;
  }
  .toggle-button {
    background-color: rgb(216, 122, 15);
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
    color: #fff;
  }
</style>