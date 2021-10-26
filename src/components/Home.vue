<template>
  <el-container>
    <!-- 页面头部 -->
    <el-header>
      <div>
        <img src="../assets/heima.png"
             alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info"
                 @click="loginInfo">退出</el-button>
    </el-header>
    <!-- 页面主体 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse?'64px':'200px'">
        <!-- 折叠 -->
        <div class="toggle-button"
             @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单区域 -->
        <el-menu background-color="#333744"
                 text-color="#fff"
                 active-text-color="#409BFF"
                 unique-opened
                 :collapse="isCollapse"
                 :collapse-transition="false"
                 router
                 :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id+''"
                      v-for="(item) in menuList"
                      :key="item.id">
            <!-- 一级菜单模板区 -->
            <template slot="title">
              <!-- 一级菜单图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 一级菜单文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' + chitem.path"
                          v-for="(chitem) in item.children"
                          :key="chitem.id"
                          @click="saveNavState('/' + chitem.path)">
              <!-- 二级菜单模板区 -->
              <template slot="title">
                <!-- 二级菜单图标 -->
                <i class="el-icon-menu"></i>
                <!-- 二级菜单文本 -->
                <span>{{chitem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      //左侧菜单数据
      menuList: [],
      //图标对象
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      //是否折叠
      isCollapse: false,
      //被激活的链接地址
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 退出到登录页
    loginInfo () {
      // 清空 token
      window.sessionStorage.clear()
      // 重定向到 login
      this.$router.push('/login')
    },
    //获取所有的菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.message.error(res.meta.msg)
      this.menuList = res.data
    },
    //点击折叠菜单栏
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    //保存链接的激活状态
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
};
</script>

<style lang="less" scoped>
.el-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0;
  font-size: 20px;
  color: #fff;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 10px;
    }
  }
}

.el-aside {
  background-color: #333744;
  .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    color: #fff;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
</style>