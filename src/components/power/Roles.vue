<template>
    <div>
      <!-- 面包屑导航区域 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!-- 卡片视图 -->
      <el-card>
        <!-- 添加角色按钮区域 -->
        <el-row>
          <el-col>
            <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
          </el-col>
        </el-row>

        <!-- 角色列表区域 -->
        <el-table :data="roleslist" border stripe row-key="id">
          <!-- 展开列 -->
          <!--加上type=expand即可-->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id" :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']">
                <!-- 渲染一级权限 -->
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)">
                    {{item1.authName}}
                  </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 渲染二级和三级权限 -->
                <el-col :span="19">
                  <!-- 通过 for 循环 嵌套渲染二级权限 -->
                  <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" >
                    <el-col :span="6">
                      <el-tag type='success' closable @close="removeRightById(scope.row, item2.id)">
                        {{item2.authName}}
                      </el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
              <!--<pre>
                {{scope.row}}
              </pre>-->
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
            <!--作用域插槽-->
            <!--scope是自定义属性值名，其下有row的属性-->
            <template slot-scope="scope">
              <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeUserById(scope.row.id)">删除</el-button>
              <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
</template>
</el-table-column>
</el-table>
      </el-card>

      <!-- 添加用户的对话框 -->
      <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <!-- 对话框主体区域 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
          <!--prop 校验规则-->
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 对话框底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户的对话框 -->
      <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editForm" ref="editFormRef" label-width="70px">
          <el-form-item label="角色 ID">
            <el-input v-model="editForm.roleId" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserInfo">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配权限的对话框 -->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
            <!--show-checkbox:显示复选框
            node-key:设置选中节点对应的值
            default-expand-all:是否默认展开所有节点
            :default-checked-keys 设置默认选中项的数组
            ref:设置引用 -->
        <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
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
        // 所有角色列表
        roleslist: [],

        // 控制添加用户对话框的显示与隐藏
        addDialogVisible: false,
        // 添加用户的表单数据
        addForm: {
          roleName: '',
          roleDesc: ''
        },
        // 添加表单的验证规则对象
        addFormRules: {
          roleName: [
            { required: true, message: '请输入角色名称', trigger: 'blur' }
          ],
          roleDesc: [
            { required: true, message: '请输入角色描述', trigger: 'blur' }
          ]
        },

        // 控制修改用户对话框的显示与隐藏
        editDialogVisible: false,
        // 查询到的用户信息对象
        editForm: {},
        // 修改表单的验证规则对象
        editFormRules: {
          roleName: [
            { required: true, message: '请输入角色名称', trigger: 'blur' }
          ],
          roleDesc: [
            { required: true, message: '请输入角色描述', trigger: 'blur' }
          ]
        },

        // 控制分配权限对话框的显示与隐藏
        setRightDialogVisible: false,
        // 所有权限的数据
        rightslist: [],
        // 树形控件的属性绑定对象
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        // 默认选中的节点Id值数组
        defKeys: [],
        // 当前即将分配权限的角色id
        roleId: ''
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      // 获取所有角色列表
      async getRolesList() {
        const { data: res } = await this.$http.get('/roles')
        if (res.meta.status !== 200) return this.$message.error('获取角色列表失败!')
        this.roleslist = res.data
        // console.log(this.roleslist)
      },

      // 监听添加用户对话框的关闭事件
      addDialogClosed() {
        // 对话框关闭之后，重置表达
        // 重置表单需要找到ref的引用对象，即addFormRef
        // resetFields对整个表单进行重置，将所有字段值重置为初始值并移除校验结果
        // console.log(this)
        this.$refs.addFormRef.resetFields()
      },
      // 点击按钮，添加新用户
      addUser() {
        // 调用validate进行表单验证
        // 欲校验表单需要找到ref的引用对象，即addFormRef
        this.$refs.addFormRef.validate(async valid => {
          // console.log(valid)
          if (!valid) return this.$message.error('请填写完整用户信息')
          // 可以发起添加用户的网络请求
          const { data: res } = await this.$http.post('roles', this.addForm)
          // 判断如果添加失败，就做提示
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败！')
          }
          // 添加成功的提示
          this.$message.success('添加用户成功！')
          // 隐藏添加用户的对话框
          this.addDialogVisible = false
          // 重新获取用户列表数据
          this.getRolesList()
        })
      },

      // 展示编辑用户的对话框
      async showEditDialog(id) {
        const { data: res } = await this.$http.get('roles/' + id)
        if (res.meta.status !== 200) return this.$message.error('查询用户失败')
        this.editForm = res.data
        this.editDialogVisible = true
        // console.log(id)
      },
      // 监听修改用户对话框的关闭事件
      editDialogClosed() {
        // 对话框关闭之后，重置表达
        // 重置表单需要找到ref的引用对象，即editFormRef
        // resetFields对整个表单进行重置，将所有字段值重置为初始值并移除校验结果
        // console.log(this)
        this.$refs.editFormRef.resetFields()
      },
      // 修改用户信息并提交
      editUserInfo() {
        // 调用validate进行表单验证
        // 欲校验表单需要找到ref的引用对象，即addFormRef
        this.$refs.editFormRef.validate(async valid => {
          // console.log(valid)
          if (!valid) return this.$message.error('请填写完整用户信息')
          // 可以发起添加用户的网络请求
          const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, this.editForm)
          // 判断如果修改失败，就做提示
          if (res.meta.status !== 200) {
            this.$message.error('修改用户失败！')
          }
          // 修改成功的提示
          this.$message.success('修改用户成功！')
          // 隐藏修改用户的对话框
          this.editDialogVisible = false
          // 重新获取用户列表数据
          this.getRolesList()
        })
      },

      // 根据Id删除对应的用户信息
      async removeUserById(id) {
        // console.log(id)
        // 弹框询问用户是否删除数据
        const confirmResult = await this.$confirm('请问是否要永久删除该角色?', '提示', {
          confirmButtonText: '确定删除',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        // console.log(confirmResult)
        // 如果用户确认删除，则返回值为字符串 confirm
        // 如果用户取消了删除，则返回值为字符串 cancel
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        // 发送请求根据id完成删除操作
        const { data: res } = await this.$http.delete('roles/' + id)
        // 判断,如果删除失败，就做提示
        if (res.meta.status !== 200) {
          return this.$message.error('删除角色失败！')
        }
        // 修改成功的提示
        this.$message.success('删除角色成功！')
        // 重新请求最新的数据
        this.getRolesList()
      },
      // 根据Id删除对应的权限
      async removeRightById(role, rightId) {
        // 弹框提示用户是否要删除
        const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(error => error)

        if (confirmResult !== 'confirm') return this.$message.info('取消了删除！')
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) return this.$message.error('删除权限失败！')
        // 无需再重新加载所有权限
        // 只需要对现有的角色权限进行更新即可
        // this.getRolesList()
        role.children = res.data
      },
      // 展示分配权限的对话框
      async showSetRightDialog(role) {
        this.roleId = role.id
        // 获取所有权限的数据
        const { data: res } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限数据失败！')
        }
        // 把获取到的权限数据保存到 data 中
        this.rightslist = res.data
        // 调用getLeafKeys进行递归，将三级权限添加到数组中
        this.getLeafKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      },
      // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
      getLeafKeys(node, arr) {
        // 如果当前 node 节点不包含 children 属性，则是三级节点
        if (!node.children) {
          return arr.push(node.id)
        }
        // 递归调用
        node.children.forEach(item => this.getLeafKeys(item, arr))
      },
      // 监听分配权限对话框的关闭事件
      setRightDialogClosed() {
        this.defKeys = []
      },
      // 点击为角色分配权限
      async allotRights() {
        // 当用户在树形权限对话框中点击确定，将用户选择的
        // //权限发送请求进行更新

        // 获取所有选中及半选的内容
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        // 将数组转换为 , 拼接的字符串
        const idStr = keys.join(',')
        // 发送请求完成更新
        const { data: res } = await this.$http.post(
          `roles/${this.roleId}/rights`,
          { rids: idStr }
        )

        if (res.meta.status !== 200) {
          return this.$message.error('分配权限失败！')
        }
        this.$message.success('分配权限成功！')
        this.getRolesList()
        // 关闭对话框
        this.setRightDialogVisible = false
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
