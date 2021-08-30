<template>
  <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>添加商品</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片视图 -->
<el-card>
  <!-- 提示区域 -->
  <el-alert
    title="在这里添加商品信息哦~"
    type="warning"
    show-icon :closable="false">
  </el-alert>
  <!-- 步骤条 -->
  <el-steps :space="250" :active="activeIndex - 0" finish-status="success" align-center>
  <el-step title="基本信息"></el-step>
  <el-step title="商品参数"></el-step>
  <el-step title="商品属性"></el-step>
  <el-step title="商品图片"></el-step>
  <el-step title="商品内容"></el-step>
  <el-step title="完成✌"></el-step>
</el-steps>
<!-- tab栏 -->
<el-form :model="addForm" :rules="addFormRules" ref="addFormRef"
 label-width="100px" class="demo-ruleForm" label-position="top">
 <el-tabs :tab-position="'left'"  v-model="activeIndex"
 :before-leave="beforeTabLeave" @tab-click="tabClicked">
    <el-tab-pane label="基本信息" name="0">
      <el-form-item label="商品名称" prop="goods_name">
        <el-input v-model="addForm.goods_name"></el-input>
      </el-form-item>
       <el-form-item label="商品价格" prop="goods_price">
        <el-input v-model="addForm.goods_price" type="number"></el-input>
      </el-form-item>
       <el-form-item label="商品重量" prop="goods_weight">
        <el-input v-model="addForm.goods_weight" type="number"></el-input>
      </el-form-item>
       <el-form-item label="商品数量" prop="goods_number">
        <el-input v-model="addForm.goods_number" type="number"></el-input>
      </el-form-item>
      <el-form-item label="商品分类" prop="goods_cat">
         <el-cascader
         expand-trigger="hover"
        v-model="addForm.goods_cat"
        :options="catelist"
        :props="cateProps"
        @change="handleChange"></el-cascader>
      </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="配置管理" name="1">
      <!-- 渲染表单的item项 -->
      <el-form-item :label="item.attr_name" v-for="item in manyTableData"
       :key="item.attr_id">
       <!-- 复选框组 -->
        <el-checkbox-group v-model="item.attr_vals" >
    <el-checkbox border :label="cb" v-for="(cb,i) in item.attr_vals"
    :key="i"></el-checkbox>
  </el-checkbox-group>
       </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品属性" name="2">
      <el-form-item :label="item.attr_name" v-for="item in onlyTableData"
      :key="item.attr_id">
      <el-input v-model="item.attr_vals"></el-input>
      </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品图片" name="3">
      <!-- action 表示图片要上传的后台API地址 -->
      <el-upload
        :headers="headersObj"
        action="https://lianghj.top:8888/api/private/v1/upload"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        list-type="picture" :on-success="handleSuccess">
  <el-button size="small" type="primary">点击上传</el-button>
  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
</el-upload>
    </el-tab-pane>
    <el-tab-pane label="商品内容" name="4">
      <!-- 富文本编辑器 -->
      <quill-editor v-model="addForm.goods_introduce">
  </quill-editor>
  <!-- 添加商品的按钮 -->
  <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
    </el-tab-pane>
  </el-tabs>
</el-form>
</el-card>

<!-- 图片预览 -->
<el-dialog
  title="图片预览"
  :visible.sync="previewVisible"
  width="50%">
  <img :src="previewPath" alt="" class="previewImg">
</el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      activeIndex: '0',
      //添加商品的表单的数据对象
      addForm:{
        goods_name: '',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        //商品所属的分类数组
        goods_cat:[],
        //图片的数组
        pics:[],
        goods_introduce:'',
        attrs:[]
      },
      addFormRules:{
        goods_name:[
           { required: true, message: '请输入商品名称~',
            trigger: 'blur' },
        ],
        goods_price:[
           { required: true, message: '请输入商品价格~',
            trigger: 'blur' },
        ],
        goods_weight:[
           { required: true, message: '请输入商品重量~',
            trigger: 'blur' },
        ],
         goods_number:[
           { required: true, message: '请输入商品数量~',
            trigger: 'blur' },
        ],
        goods_cat:[
           { required: true, message: '请选择商品分类~',
            trigger: 'blur' },
        ]
      },
      //商品分类列表
      catelist:[],
      cateProps:{
        label:'cat_name',
        value:'cat_id',
        children:'children'
      },
      //动态参数列表数据
      manyTableData:[],
      //静态参数列表数据
      onlyTableData:[],
      //图片上传的headers请求头对象
      headersObj:{
        Authorization:window.sessionStorage.getItem('token')
      },
      previewPath:'',
      previewVisible:false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取所有商品分类数据
    getCateList() {
      axios.get('https://lianghj.top:8888/api/private/v1/categories')
      .then(res => {
        if(res.data.meta.status !== 200) {
          return this.$message.error('获取商品分类数据失败~')
        }
        this.catelist = res.data.data
        // console.log(this.catelist);
      })
    },
    //级联选择器选中变化会触发这个函数
    handleChange() {
      console.log(this.addForm.goods_cat);
      if(this.addForm.goods_cat.length !== 3){
        this.addForm.goods_cat = []
        this.$message.warning('请选择至三级分类~')
      }
    },
    beforeTabLeave(activaName, oldActivaName) {
      if(oldActivaName === '0'&& this.addForm.goods_cat.length!==3){
        this.$message.error('请先选择商品分类~')
        return false
      }
    },
    tabClicked() {
      //证明访问的是动态参数面板
      if(this.activeIndex === '1') {
        // console.log(11111);
        axios.get(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes`,{params:{
          sel:'many'
        }}).then(res => {
          if(res.data.meta.status !== 200) {
            return this.$message.error('获取动态参数列表失败~')
          }
          // console.log(res.data.data);
          res.data.data.forEach(item => {
           item.attr_vals = item.attr_vals.length === 0?[] 
            : item.attr_vals.split(' ')
          })
          this.manyTableData = res.data.data
        })
      } else if(this.activeIndex === '2') {
          axios.get(`https://lianghj.top:8888/api/private/v1/categories/${this.cateId}/attributes`,{params:{
          sel:'only'
        }}).then(res => {
          if(res.data.meta.status !== 200) {
            return this.$message.error('获取静态属性失败~')
          }
          this.onlyTableData = res.data.data
        })
      }
    },
    //处理图片的预览效果
    handlePreview(file) {
    //  console.log(file);
     var url = `https://lianghj.top:8888/${file.response.data.tmp_path}`
     this.previewPath = url
     this.previewVisible = true
    },
    //处理移除图片的操作
    handleRemove(file) {

       const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x => {
        x.pic === filePath
      })
      this.addForm.pics.splice(i,1)
      // console.log(this.addForm);
      this.$message.success('图片移除成功~')
    },
    //监听图片上传成功的事件
    handleSuccess(response) {
      // console.log(response);
      //1.拼接到一个图片信息对象、
      const picInfo = {pic:response.data.tmp_path}
      //2.将图片信息对象，push到pics数组中
      this.addForm.pics.push(picInfo)
      // console.log(this.addForm)
    },
    //添加商品
    add() {
      // console.log(this.addForm.goods_introduce);
      this.$refs.addFormRef.validate(valid => {
        if(!valid) {
          return this.$message.error('请填写必要的表单项~~~')
        }
        //执行添加的业务逻辑
        const addFormClone = JSON.parse(JSON.stringify(this.addForm))
        addFormClone.goods_cat = addFormClone.goods_cat.join(',')
        //处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo ={attr_id:item.attr_id,attr_value:item.attr_vals.join(' ')}
          this.addForm.attrs.push(newInfo)
        })
        //处理静态属性
        this.onlyTableData.forEach(item =>{
          const newInfo = {attr_id:item.attr_id,attr_value:item.attr_vals}
           this.addForm.attrs.push(newInfo)
        })
        addFormClone.attrs = this.addForm.attrs
        console.log(addFormClone);

        //发起请求添加商品
        //商品的数据必须是唯一的
        axios.post('https://lianghj.top:8888/api/private/v1/goods',addFormClone).then(res => {
          if(res.data.meta.status !== 201) {
            return this.$message.error('添加商品失败了~')
          }
          this.$message.success('添加商品成功了~~~~ 😀')
          this.$router.push('/goods')
        })
      })
    }
  },
  computed:{
    cateId() {
      if(this.addForm.goods_cat.length === 3){
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang='less' scoped>
  .el-checkbox {
    margin: 0 10px 0 0 !important;
  }
  .previewImg {
    width: 100%;
  }
  .btnAdd {
    margin-top:18px ;
  }
</style>