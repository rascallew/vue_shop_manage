<template>
  <div>
     <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>参数列表</el-breadcrumb-item>
</el-breadcrumb>
<!-- 卡片视图区域 -->
<el-card>
  <!-- 头部的警告区域 -->
   <el-alert
    title="各位客官注意：只允许为第三级分类设置相关的参数哦~"
    type="warning" :closable="false"  show-icon>
  </el-alert>
  <el-row class="cat_opt">
    <span>选择商品分类：</span>
     <el-cascader
    v-model="seletedCateKeys"
    :options="catelist"
    expand-trigger="hover"
    :props="cateProps"
    @change="handleChange"></el-cascader>
  </el-row>
  <!-- tabs 页签区域 -->
   <el-tabs v-model="activeName" @tab-click="handleTabClick">
    <el-tab-pane label="动态参数" name="many">
      <el-button type="primary" size="mini" :disabled="isBtnDisabled"
      @click="addDialogVisible = true">添加参数</el-button>
      <!-- 动态参数表格 -->
      <el-table :data="manyTableData" border stripe>
        <!-- 展开行 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-tag class="tags" v-for="(item,i) in scope.row.attr_vals" :key="i" :closable="true"
            @close="handleClosed(i, scope.row)">
              {{item}}
            </el-tag>
            <!-- 输入文本框 -->
            <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)"
            >
            </el-input>
            <!-- 添加按钮 -->
            <el-button v-else class="button-new-tag" size="small" 
            @click="showInput(scope.row)">+ New Tag</el-button>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
         <el-table-column label="参数名称" prop="attr_name"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button type="primary" icon="el-icon-edit" size="mini"
               @click="showEditDialog(scope.row.attr_id) ">编辑</el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini"
              @click="removeParams(scope.row.attr_id)">删除</el-button>
            </template>
          </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="静态属性" name="only">
       <el-button type="warning" size="mini"  :disabled="isBtnDisabled"
       @click="addDialogVisible = true">添加属性</el-button>
       <!-- 静态属性表格 -->
        <el-table :data="onlyTableData" border stripe>
        <!-- 展开行 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-tag class="tags" v-for="(item,i) in scope.row.attr_vals" :key="i" :closable="true"
            @close="handleClosed(i, scope.row)">
              {{item}}
            </el-tag>
            <!-- 输入文本框 -->
            <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)"
            >
            </el-input>
            <!-- 添加按钮 -->
            <el-button v-else class="button-new-tag" size="small" 
            @click="showInput(scope.row)">+ New Tag</el-button>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
         <el-table-column label="属性名称" prop="attr_name"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button type="primary" icon="el-icon-edit" size="mini" 
              @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini"
               @click="removeParams(scope.row.attr_id)">删除</el-button>
            </template>
          </el-table-column>
      </el-table>
    </el-tab-pane>
  </el-tabs>
</el-card>

<!-- 添加参数的对话框 -->
<el-dialog
  :title="'添加'+titleText"
  :visible.sync="addDialogVisible"
  width="50%" @close="addDialogClosed">
  <!-- 添加参数的表单 -->
  <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改参数对话框 -->
<el-dialog
  :title="'修改'+titleText"
  :visible.sync="editDialogVisible"
  width="50%" @close="editDialogClosed">
  <!-- 添加参数的表单 -->
  <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>

  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return{
      //商品分类列表
      catelist:[],
      cateProps:{
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      //级联选择框双向绑定到的数组
      seletedCateKeys:[],
      //被激活的页签的名称
      activeName:'many',
      manyTableData:[],
      onlyTableData:[],
      //控制添加对话框的显示与隐藏
      addDialogVisible:false,
      //添加参数的表单数据对象
      addForm:{
        attr_name:''
      },
      //添加表单的验证规则对象
      addFormRules: {
        attr_name:[
           { required: true, message: '请输入参数名称', trigger: 'blur' },
        ]
      },
      //控制修改对话框的显示与隐藏
      editDialogVisible:false,
      //修改的表单数据对象
      editForm:{},
       //修改表单的验证规则对象
      editFormRules: {
        attr_name:[
           { required: true, message: '请输入参数名称', trigger: 'blur' },
        ]
      },
    }
  },
  created() {
    this.getCateList()
  },
  methods:{
    getCateList() {
      axios.get('https://lianghj.top:8888/api/private/v1/categories').then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取商品失败~')
        }
        // console.log(res.data.data);
        this.catelist = res.data.data
      })
    },
    //级联选择框变化会触发这个函数
    handleChange() {
      this.getParamsData()

    },

    handleTabClick() {
      console.log(this.activeName);
      this.getParamsData()
    },
    getParamsData() {
        // console.log(this.seletedCateKeys); 
      //证明不是三级分类
      if(this.seletedCateKeys.length !== 3){
        this.seletedCateKeys = []         
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      //证明是三级分类
      //根据所选的分类id，和当前的所处的面板，获取对应的参数
      axios.get(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes`,
      {params:{sel:this.activeName}}).then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取参数失败~')
        }
       //把attr_vals变成数组
        res.data.data.forEach(item => {
          item.attr_vals = item.attr_vals ?item.attr_vals.split(' '):[]
          //控制文本框的显示与隐藏
          item.inputVisible = false
          item.inputValue = ''
        })
        //  console.log(res.data.data);
        //把动态参数的数据和静态的分开
        if(this.activeName === 'many') {
          this.manyTableData = res.data.data
        }else{
          this.onlyTableData = res.data.data
        }
      })
    },
    //监听添加对话框的关闭
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮添加参数
    addParams() {
      this.$refs.addFormRef.validate(valid => {
        if(!valid) return
        axios.post(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes`,
        {attr_name:this.addForm.attr_name,attr_sel:this.activeName}).then(res => {
          if(res.data.meta.status !== 201) {
            return this.$message.error('添加参数失败~')
          }
          this.$message.success('添加参数成功~ ^_^')
          this.addDialogVisible = false
          this.getParamsData()
        })
      })
    },
    //点击按钮，展示修改的对话框
    showEditDialog(id) {
      //查询当前参数的信息
      axios.get(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes/${id}`,{params:{attr_sel:this.activeName}})
      .then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取参数信息失败~')
        }
        
        this.editForm =res.data.data
      })
      this.editDialogVisible =true
    },
    //重置修改的表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //点击按钮修改参数
    editParams() {
      this.$refs.editFormRef.validate(valid =>{
        if(!valid) return
        axios.put(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        }).then(res => {
          if(res.data.meta.status !== 200){
            this.$message.error('修改参数失败~')
          }
          this.$message.success('修改参数成功~  ^-^')
          this.getParamsData()
          this.editDialogVisible = false
        })
      })
    },
    //根据id删除指定的参数项
    async removeParams(id){
       const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.warning('已取消删除操作~')
      }
      axios.delete(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes/${id}`)
      .then(res => {
      if (res.data.meta.status !== 200) {
              return this.$message.error('删除参数失败~')
            }
            this.$message.success('删除参数成功~ ^-^')
            this.getParamsData()
      })
     
    },
    //文本失去焦点或者摁下了回车键触发
   
    handleInputConfirm(row){
      // console.log(ok)
     //小优化
      if(row.inputValue.trim().length === 0){
        row.inputValue = ''
         row.inputVisible = false
         return
      }
      //如果没有return，说明合法
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      //发起请求保存这次操作
      axios.put(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes/${row.attr_id}`,
      {attr_name:row.attr_name,
      attr_sel:row.attr_sel,
      attr_vals:row.attr_vals.join(' ')})
      .then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('修改参数项失败了~')
        }
        this.$message.success('修改参数成功~~ ^+^')
      })
    },
    showInput(row) {
      row.inputVisible = true
      //让文本框自动获得焦点
      //$nextTick 方法的作用，就是当页面上元素被重新渲染的时候，才会指定回调函数的代码
       this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
    },
    //删除对应的参数
    handleClosed(i,row) {
      row.attr_vals.splice(i,1)
       axios.put(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes/${row.attr_id}`,
      {attr_name:row.attr_name,
      attr_sel:row.attr_sel,
      attr_vals:row.attr_vals.join(' ')})
      .then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('删除参数项失败了~')
        }
        this.$message.success('删除参数成功~~ ^+^')
      })
    }
  },
  computed: {
    //如果按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      if(this.seletedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    //当前选中三级分类的id
    cateId() {
      if(this.seletedCateKeys.length === 3){
        return this.seletedCateKeys[2]
      }
      return null
    },
    //动态jisuan标题的文本
    titleText() {
      if(this.activeName ==='many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
  .cat_opt {
    margin:15px 0;
    font-size: 18px;
    font-weight: 600;
  }
  .input-new-tag {
    width: 120px;
  }
  .tags {
    margin-right: 15px;
  }
</style>