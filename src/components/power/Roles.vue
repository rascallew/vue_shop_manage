<template>
  <div>
    <!-- 面包屑导航 -->
   <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片视图 -->
<el-card>
  <!-- 添加角色列表按钮区域 -->
  <el-row>
    <el-col>
      <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
    </el-col>
  </el-row>
  <!-- 角色列表区域 -->
    <el-table :data="rolelist" border stripe>
      <!-- 展开列 -->
      <el-table-column type="expand">
        <template v-slot='scope'>
          <el-row class="vcenter" :class="['bdbottom',
          i1 === 0?'bdtop':'']" v-for="(item1,i1) in scope.row.children" 
          :key="item1.id" >
            <!-- 渲染一级权限 -->
            <el-col :span="5">
              <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- 渲染二级和三级权限 -->
            <el-col :span="19">
              <!-- 通过for循环嵌套渲染二级权限 -->
              <el-row class="vcenter" :class="[i2===0?'':'bdtop']" v-for="(item2,i2) in item1.children" 
              :key="item2.id" >
                <el-col :span="6">
                  <el-tag closable @close="removeRightById(scope.row, item2.id)" type="success">{{item2.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="13">
                  <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable
                  @close="removeRightById(scope.row, item3.id)">
                    {{item3.authName}}
                  </el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          <!-- <pre>
            {{scope.row}}
          </pre> -->
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" >
        <template v-slot="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" 
          @click="showEditDialog(scope.row.id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini"
          @click="removeRoleById(scope.row.id)">删除</el-button>
          <el-button type="warning" icon="el-icon-setting" size="mini"
          @click="showSetRightDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
</el-card>

<!-- 添加角色的对话框 -->
<el-dialog
  title="添加角色"
  :visible.sync="addDialogVisible"
  width="50%" @close="addDialogClosed">
  <!-- 内容主体区域 -->
 <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" class="demo-ruleForm">
  <el-form-item label="名称" prop="roleName" style="font-size:12px">
    <el-input   v-model="addForm.roleName"></el-input>
  </el-form-item>
   <el-form-item label="描述" prop="roleDesc">
    <el-input v-model="addForm.roleDesc"></el-input>
   </el-form-item>
 </el-form>
 <!-- 底部区域 -->
  <template  v-slot:footer>
  <span class="dialog-footer">
    <el-button @click="addDialogVisible = false" type="warning">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
  </template>
</el-dialog>

<!-- 修改角色的对话框 -->
<el-dialog
  title="修改角色"
  :visible.sync="editDialogVisible"
  width="50%" @close="editDialogClosed">
  <!-- 内容主体区域 -->
 <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px" class="demo-ruleForm">
  <el-form-item label="名称" prop="roleName" >
    <el-input   v-model="editForm.roleName"></el-input>
  </el-form-item>
   <el-form-item label="描述" prop="roleDesc">
    <el-input v-model="editForm.roleDesc"></el-input>
   </el-form-item>
 </el-form>
 <!-- 底部区域 -->
  <template  v-slot:footer>
  <span class="dialog-footer">
    <el-button @click="editDialogVisible = false" type="warning">取 消</el-button>
    <el-button type="primary" @click="editUser">确 定</el-button>
  </span>
  </template>
</el-dialog>

<!-- 分配权限的对话框 -->
<el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="50%" @close="setRightDialogClosed">
 <!-- 树形控件 -->
 <el-tree :data="rightslist" :props="treeProps" show-checkbox
 node-key="id" default-expand-all :default-checked-keys="defKeys"
 ref="treeRef"></el-tree>
  <template v-slot:footer>
   <span  class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
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
      //所有角色列表数据
      rolelist:[],
      //控制添加角色的显示与隐藏
      addDialogVisible:false,
       //添加用户表单的数据
      addForm:{
        roleName: '',
        roleDesc:'',
      },
       //添加表单的规则对象
      addFormRules:{
         roleName:[
            { required: true, message: '请输入角色名称', trigger: 'blur' },
         ],
         roleDesc:[
            { required: true, message: '请输入角色描述', trigger: 'blur' },
         ],
      },
       //控制修改角色的对话框的显示与隐藏
      editDialogVisible:false,
      //查询到的角色信息对象
      editForm:{},
      //修改表单的验证规则对象
      editFormRules:{
        roleName:[
           { required: true, message: '请输入角色名称', trigger: 'blur' },
         ],
         roleDesc:[
             { required: true, message: '请输入角色描述', trigger: 'blur' },
         ]
      },
      //控制分配权限的对话框的显示与隐藏
      setRightDialogVisible:false,
      //所有权限的数据
      rightslist:[],
      //树形控件的属性绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      //默认选中的节点id数组
      defKeys:[],
      //当前即将分配的角色id
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    //获取所有角色的列表
    getRolesList() {
      axios.get('https://lianghj.top:8888/api/private/v1/roles').then(res =>{
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取角色列表失败~')
        }
        this.rolelist = res.data.data
        // console.log(this.rolelist);
      })
    },
    //根据id删除对应的权限
    async removeRightById(role, rightId) {
      //弹框提示用户是否删除
       const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(error => error)

        if(confirmResult !== 'confirm') {
          return this.$message.warning('取消了删除~')
        }
       axios.delete(`https://lianghj.top:8888/api/private/v1/roles/${role.id}/rights/${rightId}`).
        then( res =>{
          if(res.data.meta.status !== 200) {
            return this.$message.error('删除权限失败了+_+')
          }
          this.$message.success('成功删除权限~')
          // this.getRolesList()
          role.children = res.data.data
        })
    },
    //监听添加角色对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮，添加新角色
    addUser() {
      this.$refs.addFormRef.validate(valid =>{
        // console.log(valid);
        if(!valid) return
        //可以发起添加用户的网络请求
        axios.post('https://lianghj.top:8888/api/private/v1/roles',this.addForm).then(res =>{
          if(res.data.meta.status !== 201) {
            this.$message.error('添加角色失败了=_=')
          }
          this.$message.success('添加角色成功^-^')
          //隐藏添加用户的对话框
          this.addDialogVisible = false
          //重新刷新列表
          this.getRolesList()
        })
      })
    },
    //展示编辑用户的对话框
    showEditDialog(id) {
      // console.log(id);
      axios.get(`https://lianghj.top:8888/api/private/v1/roles/${id}`).then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('查询角色失败了')
        }
      this.editForm = res.data.data
      })
      this.editDialogVisible = true
    },
    //监听修改用户对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //修改角色信息并且提交
    editUser() {
      this.$refs.editFormRef.validate(valid=>{
        if(!valid) return
        //发起修改用户的请求
        axios.put(`https://lianghj.top:8888/api/private/v1/roles/${this.editForm.roleId}`
        ,{roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc}).then(res => {
          // console.log(res.data);
          if(res.data.meta.status !== 200) {
            return this.$message.error('修改角色失败~')
          }
          //关闭对话框
          this.editDialogVisible = false
          //刷新数据列表
          this.getRolesList()
          //提示用户修改成功
          this.$message.success('更新角色成功^-^')
        })
      })
    },
     //根据id删除对应的角色信息
    async removeRoleById(id) {
      //弹框询问用户是否删除
       const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
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
        axios.delete(`https://lianghj.top:8888/api/private/v1/roles/${id}`).then(res =>{
          if(res.data.meta.status !== 200) {
            return this.$message.error('删除角色失败了')
          }
          this.$message.success('删除角色成功了哦^-^')
          this.getRolesList()
        })
    },
    //展示分配权限的对话框
    showSetRightDialog(role) {
      this.roleId = role.id
      //获取所有权限的数据
      axios.get('https://lianghj.top:8888/api/private/v1/rights/tree').then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取权限失败了~')
        }
        this.rightslist = res.data.data
        
        // console.log(this.rightslist);
      })

      //递归获取三级节点的id
      this.getLeafKeys(role,this.defKeys)

      this.setRightDialogVisible= true
    },
    //通过递归的方式，获取角色下所有的三级权限的id，并且保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      //如果当前接待你不包含children属性则是三级节点
      if(!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => {
        this.getLeafKeys(item,arr)
      })
    },
    //监听分配权限对话框关闭事件
    setRightDialogClosed() {
      //防止重复勾选
      this.defKeys = []
    },
    //点击分配权限
    allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys);
      const idStr = keys.join(',')
      axios.post(`https://lianghj.top:8888/api/private/v1/roles/${this.roleId}/rights`
      ,{rids:idStr}).then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('分配权限失败~')
        }
        this.$message.success('分配权限成功')
        this.getRolesList()
        this.setRightDialogVisible = false
      })

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