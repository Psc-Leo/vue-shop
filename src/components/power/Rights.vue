<template>
    <div>
      <!-- 面包屑导航区域 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>权限列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!-- 卡片视图 -->
      <el-card>
        <el-table :data="rightsList" border stripe>
          <el-table-column type="index"></el-table-column>
          <el-table-column label="权限名称" prop="authName"></el-table-column>
          <el-table-column label="路径" prop="path"></el-table-column>
          <el-table-column label="权限等级" prop="level">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level === '0'">一级</el-tag>
              <el-tag type="success" v-else-if="scope.row.level === '1'">二级</el-tag>
              <el-tag type="warning" v-else>三级</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
    </div>
</template>

<script>
  export default {
    data() {
      return {
        // 权限列表
        rightsList: []
      }
    },
    created() {
      // 获取所有的权限
      this.getRightsList()
    },
    methods: {
      // 获取权限列表
      async getRightsList() {
        const { data: res } = await this.$http.get('rights/list')
        // console.log(res)
        // 如果返回状态为异常状态则报错并返回
        if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
        // 如果返回状态正常，将请求的数据保存在data中
        this.rightsList = res.data
        // console.log(this.rightsList)
      }
    }
  }
</script>

<style lang="less" scoped>

</style>
