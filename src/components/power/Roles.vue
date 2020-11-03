<template>
 <div>
   <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
       <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
       <el-breadcrumb-item>权限管理</el-breadcrumb-item>
       <el-breadcrumb-item>权限列表</el-breadcrumb-item>
   </el-breadcrumb>
   <!-- 卡片视图区域 -->
   <el-card>
     <el-button type="primary">添加角色</el-button>
      <!-- 卡片视图区域 -->
     <el-table :data="RolesList" border stripe>
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
               <el-row :class="['bobottom',i1 === 0 ? 'bdtop':'','vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                 <!-- 渲染一级权限 -->
                 <el-col :span="5">
                   <el-tag @close="removeRightById(scope.row, item1.id)" closable>{{item1.authName}}</el-tag>
                   <i class="el-icon-caret-right"></i>
                 </el-col>
                 <!-- 渲染二级和三级权限 -->
                 <el-col :span="19">
                   <!-- 通过for循环嵌套渲染二级权限 -->
                   <el-row :class="[i2 === 0 ? '':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                     <el-col :span="6">
                       <el-tag type="success" @close="removeRightById(scope.row, item2.id)" closable>{{item2.authName}}</el-tag>
                       <i class="el-icon-caret-right"></i>
                     </el-col>
                     <el-col :span="18">
                       <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" @close="removeRightById(scope.row, item3.id)" closable>{{item3.authName}}</el-tag>
                     </el-col>
                   </el-row>
                 </el-col>
               </el-row>
            </template>
          </el-table-column>
            <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
               <!-- 编辑按钮 -->
               <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
               <!-- 删除按钮 -->
               <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
               <!-- 分配角色按钮 -->
              <el-tooltip  effect="dark" content="分配角色" placement="top" :enterable="false">
                 <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
              </el-tooltip>
            </template>
          </el-table-column>
     </el-table>
  </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
     title="提示"
     :visible.sync="setRightDialogVisible"
     width="50%" @close="setRightDialogClosed">
     <el-tree :data="rightslist" :props="treeProps" ref="treeRef" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys"></el-tree>
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
      // 所有角色列表数据
      RolesList: [],
      //  控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限数据
      rightslist: [],
      // 树形控件的属性绑定
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defkeys: [105, 116],
      // 当前分配权限的角色ID
      roleId: ''
    }
  },
  created() {
    //  获取所有用户权限
    this.getRolesList()
  },
  methods: {
    // 获取权限列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      this.RolesList = res.data
    },
    // 根据ID删除对应权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件,是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      // this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      // 把获取到的权限数据保存到data中
      this.rightslist = res.data
      // 递归获取三级节点的ID
      this.getLeafkeys(role, this.defkeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限id,并保存到defkeys数组中
    getLeafkeys(node, arr) {
      // 如果当前node节点不包括children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafkeys(item, arr))
    },
    setRightDialogClosed() {
      this.defkeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('分配权限失败！')
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}

</script>
<style lang="scss" scoped>
   .el-card {
    margin-top: 10px;
    .bdtop {
      border-top: 1px solid #eee;
    }
    .bobottom {
      border-bottom: 1px solid #eee;
    }
    .el-tag {
      margin: 7px;
    }
    .vcenter {
      display: flex;
      align-items: center;
    }
  }
</style>
