<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>商品分类</el-breadcrumb-item>
</el-breadcrumb>
<!-- 卡片视图 -->
<el-card>
  <el-row>
    <el-col>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
    </el-col>
  </el-row>
  <!-- 表格 -->
   
  <tree-table class="treeTable" :data='catelist' :columns='columns'
  :selection-type="false" :expand-type="false" show-index index-text="#"
  border :show-row-hover="false">
  <!-- 是否有效 -->
  <template slot="isok" slot-scope="scope">
    <i class="el-icon-success" 
    v-if="scope.row.cat_deleted === false" style="color:lightgreen"></i>
    <i v-else class="el-icon-error" style="color:red"></i>
  </template>
  <!-- 排序 -->
  <template slot="order" slot-scope="scope">
    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
    <el-tag type="warning" size="mini" v-else>三级</el-tag>
  </template>
  <!-- 操作 -->
  <template slot="opt" slot-scope="scope">
    <el-button type="primary" icon="el-icon-edit" size="mini"
     @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
     <el-button type="danger" icon="el-icon-delete" size="mini"
     @click="removeRoleById(scope.row.cat_id)">删除</el-button>
  </template>
  </tree-table>
  <!-- 分页区域 -->
   <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
</el-card>

<!-- 添加分类的对话框 -->
<el-dialog
  title="添加分类"
  :visible.sync="addcateDialogVisible"
  width="50%" @close="addCateDialogClosed">
  <!-- 添加分类的表单 -->
  <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item label="分类名称：" prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父级分类：" >
    <!-- props 用来指定配置对象 -->
     <el-cascader
    v-model="slectedKeys"
    :options="parentCateList"
    expand-trigger="hover"
    :props="cascaderProps"
    @change="parentCateChanged" clearable change-on-select></el-cascader>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addcateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改分类的对话框 -->
<el-dialog
  title="修改分类"
  :visible.sync="editCateDialogVisible"
  width="50%" @close="editCateDialogClosed">
  <el-form :model="editForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="70px" class="demo-ruleForm">
  <el-form-item label="名称" prop="cat_name">
    <el-input v-model="editForm.cat_name" ></el-input>
  </el-form-item>
  </el-form>
  <template v-slot:footer>
   <span  class="dialog-footer">
    <el-button @click="editCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editCate">确 定</el-button>
  </span>
  </template>
 
</el-dialog>
  </div>
</template>
<script>
import axios from 'axios'
export default {
  data() {
    return{
      //商品分类的数据列表，默认为空
      catelist:[],
      //查询条件
      queryInfo: {
        type:3,
        pagenum:1,
        pagesize:5
      },
      //总数据条数
      total:0,
      //为table指定列的定义
      columns:[
        {
          label:'分类名称',
          prop: 'cat_name',
        },
         {
          label:'是否有效',
          //表示将当前列定义为模板列
          type:'template',
          //表示当前这一列使用的模板名称
          template:'isok'
        },
         {
          label:'排序',
          //表示将当前列定义为模板列
          type:'template',
          //表示当前这一列使用的模板名称
          template:'order'
        },
         {
          label:'操作',
          //表示将当前列定义为模板列
          type:'template',
          //表示当前这一列使用的模板名称
          template:'opt'
        },
      ],
      //控制添加分类的显示与隐藏
      addcateDialogVisible:false,
      //添加分类表单的数据对象
      addCateForm:{
        cat_name:'',
        //父级分类的id
        cat_pid: 0,
        //分类的等级，默认是0，一级分类
        cat_level:0

      },
      //添加分类表单的验证规则对象
      addCateFormRules:{
        cat_name:[
            { required: true, message: '请输入分类名称', trigger: 'blur' },
        ]
      },
      //父级分类的列表
      parentCateList:[],
      //指定级联选择器的配置对象
      cascaderProps:{
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //选中分类的父级id数组
      slectedKeys:[],

      editCateDialogVisible:false,
      editForm:{},
      editCateFormRules:{
        cat_name:[
            { required: true, message: '分类名称不能为空', trigger: 'blur' },
        ]
      },
    }
  },
  created() {
    this.getCateList()
  },
  methods:{
    //获取商品分类数据
    getCateList() {
      axios.get('https://lianghj.top:8888/api/private/v1/categories'
      ,{params:this.queryInfo}).then(res=>{
        // console.log(res.data.data.result)
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取商品列表失败~')
        }
        this.catelist = res.data.data.result
        this.total = res.data.data.total
      })
    },
    //监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    //监听 pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    //点击按钮，展示对话框
    showAddCateDialog() {
      //先获取父级分类的数据列表
      this.getParentCateList()
      this.addcateDialogVisible = true
    },
    //获取父级分类的数据列表
    getParentCateList() {
      axios.get('https://lianghj.top:8888/api/private/v1/categories',
      {params:{type:2}}).then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取父级分类失败~')
        }
        // console.log(res.data);
        this.parentCateList = res.data.data
      })
    },
    //选择项发生变化出发这个函数
    parentCateChanged() {
      // console.log(this.slectedKeys);
      //如果 slectedKeys 数组中 的length大于0，证明选中的父级分类
      //反之，就说明没有选中任何父级分类
      if(this.slectedKeys.length > 0){
        //父级分类的id
        this.addCateForm.cat_pid =this.slectedKeys[this.slectedKeys.length - 1]
        //为当前的分类等级赋值
        this.addCateForm.cat_level = this.slectedKeys.length
      }else{
          //父级分类的id
        this.addCateForm.cat_pid = 0
        //为当前的分类等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    addCate() {
      // console.log(this.addCateForm);
      this.$refs.addCateFormRef.validate(valid => {
        if(!valid) return
        axios.post('https://lianghj.top:8888/api/private/v1/categories',this.addCateForm).then(res => {
          if(res.data.meta.status !== 201) {
            return this.$message.error('添加分类失败了~')
          }
          this.$message.success('添加分类成功了~ ^-^')
          this.getCateList()
          this.addcateDialogVisible = false
        })
      })
    },
    //重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.slectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
     //展示编辑分类的对话框
    showEditDialog(id) {
      // console.log(id);
      axios.get(`https://lianghj.top:8888/api/private/v1/categories/${id}`).then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('查询分类失败了')
        }
      this.editForm = res.data.data
      })
      this.editCateDialogVisible = true
    },
      //监听修改分类对话框的关闭事件
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    //修改分类信息并且提交
    editCate() {
      this.$refs.editCateFormRef.validate(valid=>{
        if(!valid) return
        //发起修改用户的请求
        axios.put(`https://lianghj.top:8888/api/private/v1/categories/${this.editForm.cat_id}`
        ,{cat_name:this.editForm.cat_name}).then(res => {
          // console.log(res.data);
          if(res.data.meta.status !== 200) {
            return this.$message.error('修改分类失败~')
          }
          //关闭对话框
          this.editCateDialogVisible = false
          //刷新数据列表
          this.getCateList()
          //提示用户修改成功
          this.$message.success('更新分类成功^-^')
        })
      })
    },
       //根据id删除对应的分类信息
    async removeRoleById(id) {
      //弹框询问用户是否删除
       const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(error => error)

        //如果用户确认删除，则返回字符串 confirm
        //如果用户取消删除，则返回字符串 cancel
        // console.log(confirmResult);
        if(confirmResult !== 'confirm'){
          return this.$message.warning('已经取消删除~')
        }
        axios.delete(`https://lianghj.top:8888/api/private/v1/categories/${id}`).then(res =>{
          if(res.data.meta.status !== 200) {
            return this.$message.error('删除分类失败了')
          }
          this.$message.success('删除分类成功了哦^-^')
          this.getCateList()
        })
    },
  }
}
</script>
<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }
  .el-cascader {
    width: 100%;
  }
</style>

