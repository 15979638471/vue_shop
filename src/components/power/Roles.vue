<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card class="box-card">
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row 
              v-for="(item1,i1) in scope.row.children" 
              :key="item1.id" 
              :class="['bdbottom',i1 === 0 ? 'bdtop' : '', 'vcenter']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                  @close="removeRightsById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>

              <!-- 渲染二级权限和三级权限 -->
              <el-col :span="19">
                <el-row 
                  v-for="(item2,i2) in item1.children" 
                  :key="item2.id" 
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']">
                  <!-- 渲染二级权限 -->
                  <el-col :span="5">
                    <el-tag 
                      type="success" closable
                      @close="removeRightsById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag type="warning"
                      v-for="item3 in item2.children" 
                      :key="item3.id" closable
                      @close="removeRightsById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
              <!-- 修改 -->
              <el-button 
                type="primary" icon="el-icon-edit" size="mini" 
                @click="showEditDialog(scope.row.id)">编辑</el-button>
              <!-- 删除 -->
              <el-button 
                type="danger" icon="el-icon-delete" size="mini"
                @click="deleteRolesById(scope.row.id)">删除</el-button>
              <!-- 分配权限 -->
              <el-button 
                type="warning" icon="el-icon-setting" size="mini"
                @click="showSetRightsDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改角色信息"
      :visible.sync="editDialogVisible"
      @close="editDialogClose"
      width="50%">
      <!-- 对话框内容主体 -->
      <el-form 
        ref="editFormRef"
        :model="editForm" 
        label-width="80px"
        :rules="{roleName:[{ required: true, message: '请输入角色名称', trigger: 'blur' }]}">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEditRoles()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRishtsDialogVisible"
      width="50%">
      <!-- 对话框内容主体 -->
      <el-tree 
        :data="rightsList" 
        :props="treeProps" 
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="rightsTreeRef"></el-tree>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRishtsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRishts">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      roleList: [],
      // 修改角色对话框是否显示
      editDialogVisible: false,
      // 修改角色表单数据
      editForm: {
        roleName: '',
        roleDesc: ''
      },
      // 分配权限对话框是否显示
      setRishtsDialogVisible: false,
      // 权限列表数据
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的权限id
      defKeys: [],
      // 要分配权限的角色id
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    // 获取所有角色
    async getRoleList() {
      const {data: res} = await this.$http.get('roles')

      // console.log(res);

      if(res.meta.status !== 200)
        return this.$message.error(res.meta.msg)

      // this.$message.success(res.meta.msg)
      this.roleList = res.data
    },
    // 展示修改角色的对话框
    async showEditDialog(id) {
      this.editDialogVisible = true
      const {data: res} = await this.$http.get(`roles/${id}`)
      if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // console.log(res.data);
      this.editForm = res.data
    },
    // 修改表单对话框关闭事件
    editDialogClose() {
      this.$refs.editFormRef.resetFields();
    },
    // 保存修改角色
    async saveEditRoles() {
      // 表单验证
      this.$refs.editFormRef.validate(async valid => {
        // 保存请求
        const {data: res} = await this.$http.put(`roles/${this.editForm.roleId}`,this.editForm);
        if(res.meta.status !== 200) 
          return this.$message.error(res.meta.msg)
        this.$message.success('保存角色成功！')
        this.editDialogVisible = false
        // 重载角色列表
        this.getRoleList()
      })
    },
    // 根据id删除角色
    async deleteRolesById(id) {
      // console.log(id);
      // 弹框询问用户是否删除
      const str = await this.$confirm('此操作将永久删除该用角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(() => err);
      if(str !== 'confirm') return this.$message.info('已取消删除')

      // 进行删除操作
      const {data: res} = await this.$http.delete(`roles/${id}`)
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      this.$message.success('删除角色成功！')
      // 重载角色列表
      this.getRoleList()
    },
    // 根据id移除权限
    async removeRightsById(role, rightId) {
      // console.log(role);
      const str = await this.$confirm('你确定要移除用户的该权限吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err);

      if(str !== 'confirm') 
        return this.$message.info('取消了移除权限')

      const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      role.children = res.data
      this.$message.success('移除权限成功！')
    },
    // 显示分配角色对话框
    async showSetRightsDialog(role) {
      this.roleId = role.id
      const {data: res} = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      this.rightsList = res.data
      let arr = []
      this.getDefKeys(role, arr)
      // console.log(arr);
      this.defKeys = arr

      this.setRishtsDialogVisible = true
    },
    // 获取当前角色权限默认选中的数组
    getDefKeys(node, arr) {
      if(!node.children) {
        return arr.push(node.id)
      }
      for (const item of node.children) {
        this.getDefKeys(item, arr)
      }
    },
    // 给用户分配权限
    async setRishts() {
      const ids = [
        ...this.$refs.rightsTreeRef.getCheckedKeys(),
        ...this.$refs.rightsTreeRef.getHalfCheckedKeys()
      ]

      const rids = ids.join(',')

      // console.log(ids);
      const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids})
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      this.$message.success('分配权限成功！')

      this.setRishtsDialogVisible = false
      this.getRoleList()
    }
  }
}
</script>
 
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>