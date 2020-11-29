<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card class="box-card">
      <!-- 搜索与添加 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input 
            placeholder="请输入内容" 
            class="input-with-select" 
            v-model="queryInfo.query"
            clearable
            @clear="search">
            <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 用户列表 -->
      <el-row>
        <el-table :data="userList" stripe border fit>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column prop="username" label="姓名"></el-table-column>
          <el-table-column prop="email" label="邮箱"></el-table-column>
          <el-table-column prop="mobile" label="电话"></el-table-column>
          <el-table-column prop="role_name" label="角色"></el-table-column>
          <el-table-column label="状态" prop="mg_state">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.mg_state"
                @change="userStateChanged(scope.row)">
              </el-switch>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180px">
            <template slot-scope="scope">
              <!-- 修改 -->
              <el-button 
                type="primary" icon="el-icon-edit" size="mini" 
                @click="showEditDialog(scope.row.id)"></el-button>
              <!-- 删除 -->
              <el-button 
                type="danger" icon="el-icon-delete" size="mini"
                @click="deleteUserById(scope.row.id)"></el-button>
              <!-- 分配权限 -->
              <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                <el-button 
                  type="warning" icon="el-icon-setting" size="mini"
                  @click="showSetRoleDialog(scope.row)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-row>

      <!-- 分页 -->
      <el-row>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="queryInfo.pagenum"
          :page-sizes="[1,2,5,10,15,20]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          background>
        </el-pagination>
      </el-row>
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="dialogVisible"
      @close="dialogClose"
      width="50%">
      <!-- 对话框内容主体 -->
      <el-form 
        ref="addFormRef"
        :model="addForm" 
        label-width="80px"
        :rules="addFormRules">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入邮箱地址"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入手机号码"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisible"
      @close="editDialogClose"
      width="50%">
      <!-- 对话框内容主体 -->
      <el-form 
        ref="editFormRef"
        :model="editForm" 
        label-width="80px"
        :rules="addFormRules">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" placeholder="请输入用户名" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入邮箱地址"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入手机号码"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEditUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色的对话框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%">
      <!-- 对话框内容主体 -->
      <p>用户名：{{setRoleDialogValue.username}}</p>
      <p>当前角色名：{{setRoleDialogValue.role_name}}</p>
      <el-select v-model="selectedValue" placeholder="请选择角色">
        <el-option
          v-for="item in setRoleOption"
          :key="item.id"
          :label="item.roleName"
          :value="item.id">
        </el-option>
      </el-select>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRole">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 验证手机号的规则
    let checkMobile = (rule, value, callback) => {
      const reg = /^1\d{10}$/
      if(reg.test(value)) {
        return callback()
      }

      callback(new Error("请输入正确的手机号码"))
    }

    return {
      // 用户数据列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2,
      },
      userList: [],
      total: 0,
      // 添加对话框是否显示
      dialogVisible: false,
      // 添加表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }, 
      // 修改对话框是否显示
      editDialogVisible: false,
      // 修改表单数据
      editForm: {
        username: '',
        email: '',
        mobile: ''
      }, 
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在 3 到 10 个字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '用户名的长度在 6 到 15 个字符之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 分配角色对话框是否显示
      setRoleDialogVisible: false,
      // 分配角色对话框显示用户数据
      setRoleDialogValue: {
        id: '',
        username: '',
        role_name: ''
      },
      // 角色下拉列表
      setRoleOption: {},
      selectedValue: ''
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const loadingInstance = this.$loading.service({
        target: 'main'
      })
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200)
        return this.$message.error('获取用户列表失败！')
      this.userList = res.data.users
      this.total = res.data.total
      // console.log(res)
      loadingInstance.close()
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 page 改变的事件
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 状态改变事件的处理
    async userStateChanged(userinfo) {
      console.log(userinfo);
      const {data: res} = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`)
      if(res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error("更新用户状态失败！")
      }
      this.$message.success("更新用户状态成功！")
    },
    // 搜索事件
    search() {
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    // 添加表单对话框关闭事件
    dialogClose() {
      this.$refs.addFormRef.resetFields();
    },
    // 确定按钮事件，保存user
    saveUser() {
      this.$refs.addFormRef.validate(async valid => {
        if(!valid) return
        // 发起添加用户的请求
        const {data: res} = await this.$http.post('users',this.addForm);
        if(res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }else {
          this.total += 1
          this.$message.success('添加用户成功！')
        }
        // 隐藏对话框
        this.dialogVisible = false
        // 跳转到最后一页，显示添加的数据
        // console.log(this.lastPagenum)
        this.queryInfo.pagenum = this.lastPagenum
        this.getUserList()
      })
    },
    // 展示修改用户的对话框
    async showEditDialog(id) {
      this.editDialogVisible = true
      const {data: res} = await this.$http.get(`users/${id}`)
      if(res.meta.status !== 200) return this.$message.error('查询用户数据失败！')
      // console.log(res.data);
      this.editForm = res.data
    },
    // 修改表单对话框关闭事件
    editDialogClose() {
      this.$refs.editFormRef.resetFields();
    },
    // 保存修改的用户信息
    saveEditUser() {
      console.log(this.editForm);
      this.$refs.editFormRef.validate(async valid => {
        if(!valid) return
        // 发起添加用户的请求
        const {data: res} = await this.$http.put(`users/${this.editForm.id}`,this.editForm);
        if(res.meta.status !== 200)
          this.$message.error('修改用户失败！')

        this.$message.success('修改用户成功！')
        // 隐藏对话框
        this.editDialogVisible = false
        // 重载表格数据
        this.getUserList()
      })
    },
    async deleteUserById(id) {
      console.log(id);
      const str = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err);
      
      if(str !== 'confirm') return this.$message.info('已取消删除')
      const {data: res} = await this.$http.delete(`users/${id}`)
      if(res.meta.status !== 200) return this.$message.error('删除失败！')
      this.$message.success('删除成功！')
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    // 展示分配角色对话框
    async showSetRoleDialog(userinfo) {
      this.setRoleDialogValue.id = userinfo.id
      this.setRoleDialogValue.username = userinfo.username
      this.setRoleDialogValue.role_name = userinfo.role_name

      const {data: res} = await this.$http.get('roles')
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      this.setRoleOption = res.data
      this.setRoleDialogVisible = true
    },
    // 分配角色
    async setRole() {
      const {data: res} = await this.$http
        .put(`users/${this.setRoleDialogValue.id}/role`,{rid: this.selectedValue})
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      this.$message.success('分配角色成功！')
      this.setRoleDialogVisible = false
      this.getUserList()
    }
  },
  computed: {
    lastPagenum() {
      return Math.ceil(this.total / this.queryInfo.pagesize)
    }
  }
}
</script>

<style lang="less" scoped>
</style>