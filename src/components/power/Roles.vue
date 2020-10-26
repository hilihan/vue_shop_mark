<template>
  <div>
    <!--面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <!--添加角色按钮区-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!--角色列表区域-->
      <el-table :data="roleList" stripe border>
        <!--展开列-->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!--<pre>{{scope.row.children}}</pre>-->
            <el-row :class="['bdbottom',i1 === 0 ? 'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!--渲染一级权限-->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--渲染二级、三级权限-->
              <el-col :span="19">
                <!--通过for循环嵌套渲染二级权限-->
                <el-row :class="[i2 === 0 ? '' : 'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag closable type="success" @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable type="warning" v-for="(item3,i3) in item2.children" :key="item3.id" @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!--索引列-->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--添加角色的对话框-->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!--内容主体区-->
      <el-form :model="addRoleForm" :rules="RoleRules" ref="addRoleRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!--修改角色的对话框-->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <!--内容主体区-->
      <el-form :model="editRoleForm" :rules="RoleRules" ref="editRoleRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!--分配权限的对话框-->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!--树形控件-->
      <el-tree show-checkbox node-key="id" default-expand-all :data="rightsList" :props="treeProps"
        :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    data() {
      return {
        //所有角色列表数据
        roleList: [],
        //角色添加的表单数据
        addRoleForm: {
          roleName: '',
          roleDesc: ''
        },
        //控制添加角色对话框的显示和隐藏
        addDialogVisible: false,
        //角色的验证规则对象
        RoleRules: {
          roleName: [{
              required: true,
              message: '请输入角色名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '用户名长度在 3 到 10 个字符之间',
              trigger: 'blur'
            }
          ]
        },
        //编辑角色的表单数据
        editRoleForm: {},
        //控制编辑角色对话框的显示和隐藏
        editDialogVisible: false,
        //分配权限对话框的显示和隐藏
        setRightDialogVisible: false,
        //所有权限的数据
        rightsList: [],
        //树形控件属性绑定对象
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        //默认选中的节点Id数组
        defKeys: [],
        //当前即将分配权限的角色ID
        roleId:''
      }
    },
    created() {
      this.getRolesList();
    },
    methods: {
      //获取所有角色的列表
      async getRolesList() {
        const {
          data: res
        } = await this.$http.get('roles');
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败！' + res.meta.msg);
        }
        this.roleList = res.data;
        //console.log(this.roleList);
      },
      //添加角色
      addRole() {
        this.$refs.addRoleRef.validate(async valid => {
          if (!valid) return;
          const {
            data: res
          } = await this.$http.post('roles', this.addRoleForm);
          if (res.meta.status !== 201) {
            return this.$message.error('添加角色失败！' + res.meta.msg);
          }
          this.$message.success('添加角色成功！');
          //隐藏添加角色对话框
          this.addDialogVisible = false;
          //重新获取角色数据列表
          this.getRolesList();
        })
      },
      //监听添加角色对话框的关闭事件
      addDialogClosed() {
        this.$refs.addRoleRef.resetFields();
      },
      //展示编辑角色的对话框
      async showEditDialog(id) {
        const {
          data: res
        } = await this.$http.get('roles/' + id);
        if (res.meta.status !== 200) {
          return this.$message.error('查询用户信息失败！' + res.meta.msg);
        }
        this.editDialogVisible = true;
        this.editRoleForm = res.data;
      },
      //监听编辑角色对话框的关闭事件
      editDialogClosed() {
        this.$refs.editRoleRef.resetFields();
      },
      //修改角色信息
      editRoleInfo() {
        this.$refs.editRoleRef.validate(async valid => {
          if (!valid) return;
          //可以发起修改角色的网络请求
          const {
            data: res
          } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc
          });
          if (res.meta.status !== 200) {
            return this.$message.error('修改角色失败！' + res.meta.msg);
          }
          this.$message.success('修改角色成功！');
          //隐藏添加角色对话框
          this.editDialogVisible = false;
          //重新获取角色数据列表
          this.getRolesList();
        });
      },
      //根据角色ID删除角色信息
      async removeRoleById(id) {
        const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
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
        } = await this.$http.delete('roles/' + id);
        if (res.meta.status !== 200) {
          return this.$message.error('删除角色失败！' + res.meta.msg);
        }
        this.$message.success('删除角色成功！');

        //重新获取角色数据列表
        this.getRolesList();
      },
      //根据ID删除角色对应的权限
      async removeRightById(role, rightId) {
        //弹框提示用户是否要删除
        const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err);

        //弹框返回值为字符串 confirm 或者 cancel
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除');
        }

        //确认删除
        const {
          data: res
        } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`);
        if (res.meta.status !== 200) {
          return this.$message.error('删除角色权限失败！' + res.meta.msg);
        }

        role.children = res.data;
      },
      //展示分配权限的对话框
      async showSetRightDialog(role) {
        this.roleId = role.id;
        //获取所有权限数据
        const {data: res} = await this.$http.get('rights/tree');
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限失败！' + res.meta.msg);
        }
        this.rightsList = res.data;
        //console.log(this.rightsList);
        this.getLeafKeys(role, this.defKeys);

        this.setRightDialogVisible = true;
      },
      //通过递归的形式获取角色下所有三级权限的ID
      getLeafKeys(node, arr) {
        //如果当前node节点不包含children属性，则是三级节点，
        if (!node.children) {
          return arr.push(node.id);
        }
        node.children.forEach(item => this.getLeafKeys(item, arr));
      },
      //监听分配权限对话框的关闭事件
      setRightDialogClosed(){
         this.defKeys = [];
         this.roleId = '';
      },
      //为角色分配权限
      async allotRights(){
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ];
        //console.log(keys);
        const idStr = keys.join(',');

        const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr});
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色权限失败！' + res.meta.msg);
        }
        this.$message.success('分配角色权限成功！');
        
        this.getRolesList();
        
        this.setRightDialogVisible = false;
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
