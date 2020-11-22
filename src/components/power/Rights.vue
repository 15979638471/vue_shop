<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card class="box-card">
      <el-table :data="authList" stripe border fit>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column prop="authName" label="权限"></el-table-column>
          <el-table-column prop="path" label="路径"></el-table-column>
          <el-table-column prop="level" label="等级">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level === '1'">等级一</el-tag>
              <el-tag v-else-if="scope.row.level === '2'" type="info">等级二</el-tag>
              <el-tag v-else type="success">等级三</el-tag>
            </template>
          </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      authList: []
    }
  },
  created() {
    this.getAuthList()
  },
  methods: {
    async getAuthList() {
      const {data: res} = await this.$http.get('rights/list')
      console.log(res);
      if(res.meta.status !== 200)
        return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.authList = res.data
    }
  }
}
</script>

<style lang="less" scoped>

</style>