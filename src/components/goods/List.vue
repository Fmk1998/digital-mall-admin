<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table表格区域 -->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品ID" prop="pid"></el-table-column>
        <el-table-column label="商品名称" prop="pname"></el-table-column>
        <el-table-column label="商品价格(元)" prop="price" width="95px"></el-table-column>
        <el-table-column label="创建时间" prop="pdate" width="140px"></el-table-column>
        <el-table-column label="商品图片" prop="pimg"></el-table-column>

        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.pid)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.pid)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 添加商品的对话框 -->
      <el-dialog title="添加商品" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <!-- 内容主体区域 -->
        <el-form :model="addForm" ref="addGoodRef" label-width="70px">
          <el-form-item label="商品名" prop="pname">
            <el-input v-model="addForm.pname"></el-input>
          </el-form-item>
          <el-form-item label="商品价格" prop="price">
            <el-input v-model="addForm.price"></el-input>
          </el-form-item>
          <el-form-item label="商品图片" prop="pimg">
            <el-upload
              class="upload-demo"
              action="https://jsonplaceholder.typicode.com/posts/"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              multiple
              :limit="1"
              :file-list="fileList">
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-form-item>
          <el-form-item label="商品分类" prop="cid">
            <el-input v-model="addForm.cid"></el-input>
          </el-form-item>
          <el-form-item label="商品描述" prop="pdetail">
            <el-input v-model="addForm.pdetail"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addGoods">确 定</el-button>
      </span>
      </el-dialog>

      <!-- 修改用户的对话框 -->
      <el-dialog title="修改商品" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editForm"  ref="editFormRef" label-width="70px">
          <el-form-item label="商品名称">
            <el-input v-model="editForm.pname"></el-input>
          </el-form-item>
          <el-form-item label="商品价格">
            <el-input v-model="editForm.price"></el-input>
          </el-form-item>
          <el-form-item label="商品图片">
            <el-input v-model="editForm.pimg"></el-input>
          </el-form-item>
          <el-form-item label="商品种类">
            <el-input v-model="editForm.cid"></el-input>
          </el-form-item>
          <el-form-item label="商品详情">
            <el-input v-model="editForm.pdetail"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
      </el-dialog>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.current" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.size" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
      </el-pagination>


    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        current: 1,
        size: 10
      },
      // 商品列表
      goodslist: [],
      // 总数据条数
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        pname:"",
        price:0,
        pimg:"",
        cid:0,
        pdetail:""
      },
      // 控制修改对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      //上传图片
      fileList:[]
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get(`product?current=${this.queryInfo.current}&size=${this.queryInfo.size}`)

      // if (res.code !==0 || !res.code) {
      //   return this.$message.error('获取商品列表失败！')
      // }

      this.$message.success('获取商品列表成功！')
      console.log(res.data)
      this.goodslist = res.data.records
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.size = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.current = newPage
      this.getGoodsList()
    },
    //删除
    async removeById(pid) {
      console.log(pid)
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除！')
      }

      const { data: res } = await this.$http.delete('product?idList='+pid);


      this.$message.success('删除成功！')
      this.getGoodsList()
    },
    goAddpage() {
      this.addDialogVisible = true;
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      this.$refs.addGoodRef.resetFields()
    },
    //增加商品
    async addGoods() {
      let d = new Date();
      this.addForm.pdate = d.getFullYear()+"-"+(d.getMonth()+1)+"-"+d.getDate();
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('product', this.addForm)

        if (res.code !== 0 || !res.code) {
          this.$message.error('添加商品失败！')
        }

        this.$message.success('添加商品成功！')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户列表数据
        this.getGoodsList();
        debugger
    },
    //修改商品
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        let d = new Date();
        this.editForm.pdate = d.getFullYear()+"-"+(d.getMonth()+1)+"-"+d.getDate();
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('product',this.editForm)

        debugger
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getGoodsList()
        // 提示修改成功
        this.$message.success('更新用户信息成功！')
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog(id) {
      console.log(id)
      const { data: res } = await this.$http.get('product/' + id)


      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //上传
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handleSuccess(res,file,fileList){
      console.log(file,fileList)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
