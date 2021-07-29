<template>
    <div>
        <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>权限管理</el-breadcrumb-item>
  <el-breadcrumb-item>角色列表</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片区域 -->
<el-card>
    <!-- 添加角色按钮区域 -->
    <el-row>
        <el-col>
             <el-button type="primary" @click="addRolesVisible = true">添加角色</el-button>
        </el-col>
    </el-row>
    <!-- 角色列表区域 -->
     <el-table :data="rolesList" border stripe>
         <el-table-column type="expand">
             <template slot-scope="scope">
                 <el-row :class="['bdbottom',i1 ===0?'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                     <!-- 渲染一级权限 -->
                     <el-col :span="5">
                         <el-tag closable="" @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                         <i class="el-icon-caret-right"></i>
                     </el-col>
                     <!-- 渲染二级和三级权限 -->
                     <el-col :span="19">
                         <!-- 通过for循环嵌套渲染二级权限 -->
                         <el-row :class="[i2 === 0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                             <el-col :span="6">
                                 <el-tag type="success" closable="" @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                 <i class="el-icon-caret-right"></i>
                             </el-col>
                             <el-col :span="18">
                                 <el-tag type="warning" v-for="(item3,) in item2.children" :key="item3.id" closable="" @close="removeRightById(scope.row,item3.id)">
                                     {{item3.authName}}
                                 </el-tag>
                             </el-col>
                         </el-row>
                     </el-col>
                 </el-row>
                
             </template>
         </el-table-column>
         <el-table-column type="index"></el-table-column>
         <el-table-column label="角色名称" prop="roleName"></el-table-column>
         <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
         <el-table-column label="操作" width="300px">
             <template slot-scope="scope">
        <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoles(scope.row.id)">编辑</el-button>
        <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesByid(scope.row.id)">删除</el-button>
        <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
             </template>
         </el-table-column>
     </el-table>
</el-card>

<!-- 分配权限的对话框 -->
<el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
 <!-- 树形控件 -->
 <el-tree :data="rightsList" :props="treeProps" show-checkbox="" node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
<!-- 添加角色对话框 -->
<el-dialog title="添加角色" :visible.sync="addRolesVisible" width="50%" @close="addRolesClosed">
    <!-- 内容主体区域 -->
    <el-form :model="addRolesForm" :rules="addRolesRules" ref="addRolesRef" label-width="80px" >
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="addRolesForm.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addRolesForm.roleDesc" ></el-input>
  </el-form-item>
  
</el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addRolesVisible = false">取 消</el-button>
    <el-button type="primary" @click="addRoles">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改角色对话框 -->
<el-dialog title="修改角色" :visible.sync="editDialogRolesVisible" width="50%">
  <el-form :model="editRolesForm" :rules="editRolesFormRules" ref="editRolesFormRef" label-width="70px" >
  <el-form-item label="角色名称" >
    <el-input v-model="editRolesForm.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" >
    <el-input v-model="editRolesForm.roleDesc" ></el-input>
  </el-form-item>
  
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogRolesVisible = false">取 消</el-button>
    <el-button type="primary" @click="editRoles">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return {
            // 角色列表
            rolesList:[],
            // 控制添加用户对话框得显示与隐藏
            addRolesVisible: false,
            editRolesForm:{},
            // 添加角色的表单数据
            addRolesForm:{
                roleName:'',
                roleDesc:'',
            },
            // 添加角色验证规则
            addRolesRules:{
                roleName:{
                    required:true,
                        message:'请输入角色名',
                        trigger:'blur'
                },
                roleDesc:{
                    required:true,
                    message:'请输入角色描述',
                    trigger:'blur'
                }
                
            },
            // 修改角色验证规则
            editRolesFormRules:{
                 roleName:{
                    required:true,
                        message:'请输入角色名',
                        trigger:'blur'
                },
                roleDesc:{
                    required:true,
                    message:'请输入角色描述',
                    trigger:'blur'
                }
            },
            //控制分配权限的对话框
                setRightDialogVisible:false,
                // 所有权限的数组
                rightsList:[],
                // 树形控件的绑定对象
                treeProps:{
                    label:'authName',
                    children:'children',
                },
                // 默认选中的节点id值数组
                defKeys:[],
                //即将分配权限的id
                rolId:'',
                // 控制编辑角色对话框显示与隐藏
                editDialogRolesVisible:false
        }
    },
    created(){
        // 获取角色列表
        this.getRolesList()
    },
    methods:{
       async getRolesList(){
        const {data:res} =  await  this.$http.get('roles')
        if(res.meta.status !== 200){
            return this.$message.error('获取角色失败')
        }
        this.rolesList = res.data;
        console.log(this.rolesList);
        },
        // 根据id删除对应的权限
         async removeRightById(role,rightId){
            //弹框提示用户是否要删除
         const confirmResult =   await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err )
        if(confirmResult != 'confirm'){
            return this.$message.info('取消了删除')
        }
     const {data:res} =  await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
     if(res.meta.status !== 200){
         return this.$message.error('删除权限失败')
     }
     this.$message.success('删除权限成功')
     role.children = res.data
        },

        // 展示分配权限的对话框
       async showSetRightDialog(role){
           this.roleId = role.id
            //获取所有权限的数据
         const {data:res} = await  this.$http.get('/rights/tree')
         if(res.meta.status !== 200){
             return this.$message.error('获取权限数据失败')
         }
            // this.$message.success('获取权限数据成功')
            this.rightsList = res.data
            // 递归获取三级节点的ID
            this.getLeafKeys(role,this.defKeys)
            this.setRightDialogVisible = true
        },
        // 通过递归的形式 获取角色下所有三级权限的id 并保存到defKeys数组中
        getLeafKeys(node,arr){
            //如果当前node节点不包含children 属性，则是三级节点
            if(!node.children){
                return arr.push(node.id)
            }
            node.children.forEach(item => {
                this.getLeafKeys(item,arr)
            })
        },
        // 监听分配权限对话框的关闭事件
        setRightDialogClosed(){
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights(){
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),...this.$refs.treeRef.getHalfCheckedKeys()
            ]
           const idStr = keys.join(',')
       const {data:res} =  await  this.$http.post(`roles/${this.roleId}/rights`,{
               rids:idStr
           })
           if(res.meta.status !== 200){
               return this.$message.error('分配权限失败')
           }
           this.$message.success('分配权限成功')
           this.getRolesList()
           this.setRightDialogVisible = false
        }, 
        // 添加角色函数
        addRoles(){
            this.$refs.addRolesRef.validate(async valid => {
                 if(!valid) return
                //可以发起添加角色网络请求
               const {data:res} =  await this.$http.post('roles',this.addRolesForm)
               if(res.meta.status !==201){
                    return this.$message.error('添加角色失败')
               }
               this.$message.success('添加用户成功')
               //隐藏添加角色的对话框
              this.addRolesVisible = false;
               this.getRolesList()
            })
        },
        // 关闭添加角色对话框
        addRolesClosed(){
           this.addRolesForm = {}
        },
        // 编辑角色函数
     async showEditRoles(id){
 const {data:res} =  await this.$http.get('roles/' + id)
          if(res.meta.status !=200){
              return this.$message.error('查询角色信息失败')
          }
            this.editRolesForm = res.data
            
            this.editDialogRolesVisible = true;
            // console.log(this.editRolesForm);
      },
    //   修改角色信息并提交
    editRoles(){
        this.$refs.editRolesFormRef.validate(async valid => {
             if(!valid) return
               //发起修改用户信息得数据请求
               const {data:res} =  await this.$http.put('roles/' + this.editRolesForm.roleId,{
                   roleName:this.editRolesForm.roleName,
                   roleDesc:this.editRolesForm.roleDesc
               })
               if(res.meta.status !== 200){
                   return this.$message.error('更新角色信息失败')
               }
               this.$message.success('更新角色成功')
                this.editDialogRolesVisible = false
                this.getRolesList()
        })
    },
    // 根据id 删除角色所在行
   async removeRolesByid(id){
         // 弹框询问用户是否删除数据
          const confirmResult =  await  this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => {
            return err
        })
        // console.log(confirmResult);
        if(confirmResult !== 'confirm'){
            return this.$message.info('已取消删除')
        }
       const {data:res} =  await this.$http.delete('roles/' + id)
       if(res.meta.status !== 200){
           return this.$message.error('删除角色失败')
       }
       this.$message.success('删除角色成功')
       this.getRolesList()
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