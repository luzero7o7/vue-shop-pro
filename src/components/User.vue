<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片 -->
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="queryParams.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="140"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="130"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="150"></el-table-column>
        <el-table-column label="状态" width="70">
          <el-switch v-model="info.row.mg_state" slot-scope="info"></el-switch>
        </el-table-column>
        <!-- 操作用户按钮 -->
        <el-table-column label="操作" width="230">
          <template slot-scope="info">
            <!-- 编辑按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(info.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="delUser(info.row.id)"
            ></el-button>
            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 数据分页展示区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryParams.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryParams.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="queryParams.total"
      ></el-pagination>
      <!-- 添加用户弹窗结构 -->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
        <!-- 添加表单内容校验 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑用户的弹窗结构 -->
      <el-dialog
        title="添加用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <!-- 编辑表单内容校验 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
          <el-form-item label="用户名" prop="username" >
            <el-input v-model="editForm.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  created() {
    this.getUserInfos()
  },
  methods: {
    // 修改用户相关1
    // 展示修改用户弹窗结构
    // @param id: 被修改用户id
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
    },
    // 编辑弹窗重置清空
    editDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 编辑相关2 数据入库
    async editUser() {
      const { data: res } = await this.$http.put(
        'users/' + this.editForm.id,
        this.editForm
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editDialogVisible = false
      this.$message.success(res.meta.msg)
      this.getUserInfos()
    },
    // 删除用户
    delUser(id) {
      // 确认操作
      this.$confirm('确认删除该会员吗？', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          // 发送删除请求
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          // 提示删除成功
          this.$message.success(res.meta.msg)
          // 重载页面数据
          this.getUserInfos()
        })
        .catch(() => {})
    },

    // 进行客户端form表单校验
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // 校验成功
        if (valid) {
          const { data: res } = await this.$http.post('users', this.addForm)
          // 提交不成功
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }

          // 关闭dialog。提示成功，重载页面数据
          this.addDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    // 弹框关闭回调执行重置，清除旧内容
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 每页信息条数变化回调函数处理
    handleSizeChange(val) {
      this.queryParams.pagesize = val
      this.getUserInfos()
    },
    handleCurrentChange(val) {
      this.queryParams.pagenum = val
      this.getUserInfos()
    },
    // 获取用户列表数据
    async getUserInfos() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryParams
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      console.log(res.data.users)
      this.queryParams.total = res.data.total
      this.userInfos = res.data.users
    }
  },
  data() {
    // 手机号码自定义校验
    var checkMobile = (rule, value, callback) => {
      if (!/^1[35789]\d{9}$/.test(value)) {
        // 校验不成功
        return callback(new Error('手机号码格式不正确'))
      }
      // 校验成功 继续执行回调
      callback()
    }
    return {
      // 修改用户相关1
      // 弹窗默认不显示
      editDialogVisible: false,
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // 编辑Form表单校验规则
      editFormRules: {
        mobile: [
          { required: true, message: '手机号不可缺失', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 添加功能1
      // 添加按钮弹框是否显示
      addDialogVisible: false,
      // Form 表单参数字段
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户校验规则
      addFormRules: {
        username: [
          { required: true, message: '用户名不可缺失', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不可缺失', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机号不可缺失', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },

      // 搜索框关键字
      keywords: '',
      // 定义变量接受用户数据
      userInfos: [],
      // 定义获取数据时需要的条件参数
      queryParams: {
        query: '',
        pagenum: 1,
        pagesize: 3,
        total: 0
      }
    }
  }
}
</script>
<style lang="less" scoped>
.el-table {
  margin-top: 15px;
  font-size: 12px;
}
</style>
