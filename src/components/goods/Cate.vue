<template>
  <div>
    <!--面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图区域-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!--表格-->
      <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false"
        show-index index-text="#" border>
        <!--是否有效-->
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:#5cb6ff"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!--排序-->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!--操作-->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCateDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!--分页区域-->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>

    </el-card>

    <!--添加分类的对话框-->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%" @close="addCateDialogClosed">
      <!--添加分类的表单-->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!--options指定数据源-->
          <!--props指定配置对象-->
          <el-cascader
              v-model="selectedKeys" :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChanged" clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改商品分类的对话框-->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
      <!--内容主体区-->
      <el-form :model="editCateForm" :rules="addCateFormRules" ref="editCateFormRef" label-width="90px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>


  </div>
</template>

<script>
  export default {
    data() {
      return {
        //查询条件
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        //商品分类的数据列表
        catelist: [],
        //总数据条数
        total: 0,
        //为table指定列的定义
        columns: [{
          label: '分类名称',
          prop: 'cat_name'
        }, {
          label: '是否有效',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'isOk'
        }, {
          label: '排序',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'order'
        }, {
          label: '操作',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'opt'
        }],
        //控制添加分类对话框的显示与隐藏
        addCateDialogVisible:false,
        //添加分类的表单数据对象
        addCateForm:{
          cat_pid:0,
          cat_name:'',
          cat_level:0
        },
        //添加分类表单的的校验规则对象
        addCateFormRules:{
          cat_name:[
            { required: true,message: '请输入分类名称',trigger: 'blur'}
          ]
        },
        //父级分类的列表
        parentCateList:[],
        //指定级联选择器的配置对象
        cascaderProps:{
          value:'cat_id',
          label:'cat_name',
          children:'children',
          expandTrigger:'hover',
          checkStrictly:true
        },
        //选中的父类分类的ID数组
        selectedKeys:[],
        //控制修改分类对话框的显示与隐藏
        editCateDialogVisible:false,
        //编辑分类的表单数据对象
        editCateForm:{}
      }
    },
    created() {
      this.getCateList();
    },
    methods: {
      //获取商品分类数据
      getCateList() {
        var that = this;
        this.$http.get('categories', {
            params: this.queryInfo
          })
          .then(function(ret) {
            let data = ret.data;
            if (data.meta.status !== 200) {
              return that.$message.error('获取商品类别失败！' + data.meta.msg);
            }
            //console.log(data.data);
            //把数据列表赋值给catelist
            that.catelist = data.data.result;
            //未总数据条数保存
            that.total = data.data.total;

          }).catch(function(error) {
            console.log(error);
          });

      },
      //监听pageSize改变的事件
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize;
        this.getCateList();
      },
      //监听pagenum的改变
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage;
        this.getCateList();
      },
      //点击按钮展示添加分类对话框
      showAddCateDialog(){
        //获取父级分类的数据列表
        this.getParentCateList();

        //展示对话框
        this.addCateDialogVisible = true;
      },
      //获取父级分类的数据列表
      async getParentCateList(){
        const{data:res} = await this.$http.get('categories',{params:{type:2}});
        if(res.meta.status !== 200){
          return this.$message.error('获取父级分类数据失败！' + res.meta.msg);
        }
        //console.log(res.data);
        this.parentCateList = res.data;
      },
      //选择项发生变化触发本函数
      parentCateChanged(){
        console.log(this.selectedKeys);
        //如果 selectedKeys 数组中的 length 大于 0，说明选中了父级分类
        if(this.selectedKeys.length > 0){
          //父级分类的 ID
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1];
          //分类的等级
          this.addCateForm.cat_level = this.selectedKeys.length;
          return;
        } else {
          this.addCateForm.cat_pid = 0;
          this.addCateForm.cat_level = 0;
        }
      },
      //点击按钮添加新的分类
      addCate(){
        //console.log(this.addCateForm);
        this.$refs.addCateFormRef.validate(async valid => {
          if(!valid) return;
          const {data:res} = await this.$http.post('categories',this.addCateForm);
          if(res.meta.status !== 201){
            return this.$message.error('添加分类失败！' + res.meta.msg);
          }
          this.$message.success('添加分类成功');
          this.getCateList();
          this.addCateDialogVisible = false;
        })
      },
      //监听添加分类对话框关闭事件
      addCateDialogClosed(){
        //重置表单
        this.$refs.addCateFormRef.resetFields();
        this.selectedKeys = [];
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      },
      //监听编辑分类对话框关闭事件
      editCateDialogClosed(){
        //重置表单
        this.$refs.editCateFormRef.resetFields();
      },
      //点击按钮展示编辑分类对话框
      async showEditCateDialog(id){
        //console.log(id);
        const {data: res} = await this.$http.get('categories/' + id);
        if (res.meta.status !== 200) {
          return this.$message.error('查询分类信息失败！' + res.meta.msg);
        }
        //console.log(res.data);
        this.editCateForm = res.data;
        this.editCateDialogVisible = true;
      },
      //点击按钮修改分类
      editCate(){
        this.$refs.editCateFormRef.validate(async valid => {
          if (!valid) return;
          //可以发起修改分类的网络请求
          const {
            data: res
          } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
            cat_name: this.editCateForm.cat_name
          });
          if (res.meta.status !== 200) {
            return this.$message.error('修改分类失败！' + res.meta.msg);
          }
          this.$message.success('修改分类成功！');

          this.getCateList();

          this.editCateDialogVisible = false;
        })
      },
      //根据 ID 删除对应商品分类
      async removeCateById(id){
        const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err);

        //弹框返回值为字符串 confirm 或者 cancel
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除');
        }
        //用户确认执行删除操作
        const {
          data: res
        } = await this.$http.delete('categories/' + id);
        if (res.meta.status !== 200) {
          return this.$message.error('删除分类失败！' + res.meta.msg);
        }
        this.$message.success('删除分类成功！');

        //重新获取角色数据列表
        this.getCateList();
      }
    }
  }
</script>

<style lang="less">
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader-panel {
    height: 250px;
  }

  .el-cascader {
    width: 100%;
  }
</style>
