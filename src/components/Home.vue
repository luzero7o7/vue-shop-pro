<template>
  <el-container>
    <el-header>
      <div id="logo-box">
        <img src="../assets/img/heima.png">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside width="menyshow? '65px':'200px'">
        <div
          style="height:25px;color:white;text-align:center;linhight:25px;background-color:rgb(74,80,100);font-size:12px;letter-spacing:0.1em;user-select:none;"
          @click="menushow=!menushow"
        >|||</div>
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409EFF"
          :style="menushow?'width:65px;':'width:200px;'"
          :collapse="menushow"
          :collapse-transition="false"
          :unique-opened="true"
        >
          <el-submenu :index="item.id+''" v-for="(item,k) in menuList" :key="item.id">
            <template slot="title">
              <i :class="'iconfont icon-'+menuicon[k]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item
              :index="item.id+'-'+item2.id"
              v-for="item2 in item.children"
              :key="item2.id"
            >
              <i class="el-icon-menu"></i>
              <span>{{item2.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  // 声明周期函数
  created() {
    this.getMenuList()
  },
  data() {
    return {
      menushow: false,
      menuList: [],
      menuicon: ['user', 'tijikongjian', 'shangpin', 'danju', 'baobiao']
    }
  },
  methods: {
    // 获取左侧列表数据
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 携带token权限获得的数据赋值给menuList （数组对象形式）
      this.menuList = res.data
    },
    // 退出系统
    logout() {
      // 再次确认是否退出
      this.$confirm('是否退出?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          window.sessionStorage.removeItem('token')
          this.$router.push('/login')
        })
        .catch(() => {})
    }
  }
}
</script>

<style lang="less" scoped >
.el-container {
  height: 100%;
  .el-header {
    background-color: rgb(55, 61, 65);
    height: 60px;
    padding: 0;
    padding-right: 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    #logo-box {
      font-size: 22px;
      color: #fff;
      display: flex;
      align-items: center;
      user-select: none;
      img {
        width: 50px;
        height: 50px;
        margin-right: 10px;
      }
    }
  }
  .el-aside {
    background-color: #333744;
  }
  .el-main {
    background-color: #eaedf1;
  }
}
</style>
