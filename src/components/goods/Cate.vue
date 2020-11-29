<template>
  <div id="container">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button type="primary"
          @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <el-table
        :data="catelist"
        style="width: 100%"
        row-key="cat_id"
        border
        :tree-props="{children: 'children'}">
        <el-table-column label="#" width="80" align="center">
          <template slot-scope="scope">
            {{tableIndex[scope.row.cat_id]}}
          </template>
        </el-table-column>
        <el-table-column
          prop="cat_name" label="分类名称">
        </el-table-column>
        <el-table-column
          prop="cat_deleted" label="是否有效">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.cat_deleted"
              :active-value="false"
              :inactive-value="true">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          prop="cat_level" label="排序">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag type="danger" v-else>三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini"
              @click="deleteCateById(scope.row.cat_id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

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

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="30%"
      @close="addCateDialogClose">
      <!-- 对话框内容主体 -->
      <el-form 
        ref="addCateFormRef" 
        :model="addCateForm" 
        label-width="100px"
        :rules="formRules">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            v-model="cascaderValue"
            :options="parentCateList"
            :props="cascaderProps"
            @change="handleCascaderChange"
            style="width:100%"
            clearable></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 对话框底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 记录总条数
      total: 0,
      // 第一级的索引
      tableIndex: {},
      // 添加分类对话框是否显示
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 分类父 ID
        cat_pid: 0,
        // 分类名称
        cat_name: '',
        // 分类层级
        cat_level: 0
      },
      // 级联选择器的属性
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        checkStrictly: true
      },
      // 父级分类数据
      parentCateList: [],
      // 级联选择器选中的数据
      cascaderValue: [],
      formRules: {
        cat_name:[
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const loadingInstance = this.$loading.service({
        target: 'main'
      })
      const {data: res} = await this.$http.get('categories',{params: this.queryInfo})

      if(res.meta.status !== 200) 
        return this.$message.error('获取商品分类列表失败！')
      
      console.log(res.data)
      // 把数据列表，赋值给 catelist
      this.catelist = res.data.result
      // 记录总数据条数
      this.total = res.data.total
      // 遍历第一级商品分类
      let i = 0
      const tableIndex = {}
      for (const item of this.catelist) {
        tableIndex[item.cat_id] = ++i
      }
      this.tableIndex = tableIndex
      loadingInstance.close()
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听 page 改变的事件
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 显示添加分类对话框
    async showAddCateDialog() {
      const {data: res} = await this.$http.get('categories',{params:{type:2}})
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      // console.log(res.data);
      this.parentCateList = res.data
      this.addCateDialogVisible = true
    },
    handleCascaderChange() {
      // console.log(this.cascaderValue);
      let length = this.cascaderValue.length
      if(length > 0) {
        this.addCateForm.cat_pid = this.cascaderValue[length - 1]
      }else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      this.addCateForm.cat_level = length
      // console.log(this.addCateForm);
    },
    // 保存添加分类的方法
    addCate() {
      // console.log(this.addCateForm);
      this.$refs.addCateFormRef.validate(async valid => {
        if(!valid) return
        const {data: res} = await this.$http.post('categories',this.addCateForm)

        if(res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success('添加分类成功！')

        this.addCateDialogVisible = false
        this.getCateList()
      })
    },
    // 添加分类对话框关闭事件
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.cascaderValue = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 删除分类
    async deleteCateById(id) {
      const str = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).catch(err => err)

      if(str !== 'confirm') return

      const {data: res} = await this.$http.delete(`categories/${id}`)
      if(res.meta.status !== 200) 
        return this.$message.error(res.meta.msg)
      
      this.$message.success('删除分类成功！')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>

</style>