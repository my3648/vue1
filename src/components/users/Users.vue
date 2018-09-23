<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right" class="user-breadcrumb">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-row :gutter="20">
      <el-col :span="6">
        <el-input placeholder="请输入内容" v-model="queryStr" class="input-with-select">

          <el-button slot="append" icon="el-icon-search" @click="queryUserList"></el-button>
        </el-input>
      </el-col>
      <el-col :span="4">
        <el-button type="success" plain @click="showUserAddDialog">添加用户</el-button>
      </el-col>
    </el-row>
    <el-table :data="userList" stripe>
      <el-table-column prop="username" label="姓名" width="180">
      </el-table-column>
      <el-table-column prop="email" label="邮箱" width="180">
      </el-table-column>
      <el-table-column prop="mobile" label="电话">
      </el-table-column>
      <el-table-column prop="mg_state" label="用户状态">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.mg_state" @change="changeUserState(scope.row.id,scope.row.mg_state)"></el-switch>

        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" plain>删除</el-button>
          <el-button type="danger" plain size="mini" icon="el-icon-delete" @click="delUserById(scope.row.id)"></el-button>
          <el-button type="success" icon="el-icon-check" plain size="mini">分配角色</el-button>

        </template>
      </el-table-column>
    </el-table>

    <el-pagination background layout="prev, pager, next" :total="total" :pageSize="pageSize" :currentPage.sync="currentPage" @current-change="changePage">
    </el-pagination>

    <el-dialog title="添加用户" :visible.sync="UserAddDialog" @close="closeUserEditDialog">
      <el-form :model="UserAddForm" :rules="userAddRules" ref="UserAddForm">
        <el-form-item prop="username" label="用户名" :label-width="formLabelWidth">
          <el-input v-model="UserAddForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="password" label="密码" :label-width="formLabelWidth">
          <el-input v-model="UserAddForm.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="email" label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="UserAddForm.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="mobile" label="手机" :label-width="formLabelWidth">
          <el-input v-model="UserAddForm.mobile" autocomplete="off"></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="UserAddDialog = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>




<script>
  import axios from 'axios'
  export default {
    created() {
      this.getUserList()
    },
    data() {
      return {
        userList: [],
        // 每页条数
        pageSize: 3,
        // 当前页
        currentPage: 1,
        // 总数
        total: 1,
        queryStr: '',
        // 控制对话框显示影藏
        UserAddDialog: false,
        UserAddForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        formLabelWidth: '120px',
        userAddRules: {
          username: [
            { required: true, message: '用户名为必填项', trigger: 'blur' },
            {
              min: 3,
              max: 6,
              message: '用户名长度在 3 到 6 个字符',
              trigger: 'blur'
            }
          ],
          password: [
            { required: true, message: '密码为必填项', trigger: 'blur' },
            {
              min: 3,
              max: 6,
              message: '密码长度在 3 到 6 个字符',
              trigger: 'blur'
            }
          ]
        }
      }
    },
    methods: {
      getUserList(currentPage = 1) {
        axios
          .get('http://localhost:8888/api/private/v1/users', {
            params: {
              pagenum: currentPage,
              pagesize: 3,
              query: this.queryStr || ''
            },
            headers: {
              Authorization: localStorage.getItem('token')
            }
          })
          .then(res => {
            // console.log(res)
            const { data, meta } = res.data
            if (meta.status === 200) {
              this.userList = data.users
              this.total = data.total
              this.currentPage = data.pagenum
            }
          })
      },
      changePage(currentPage) {
        // console.log('当前页' + currentPage)
        this.getUserList(currentPage)
      },
      queryUserList() {
        // console.log(this.queryStr)
        this.getUserList()
      },

      changeUserState(id, curState) {
        axios.put(`/users/${id}/state/${curState}`).then(res => {
          console.log(res.data.data.mg_state)
          this.$message({
            type: 'success',
            message: res.data.data.mg_state === 0 ? '禁用成功' : '启用成功',
            duration: 1000
          })
        })
      },
      showUserAddDialog() {
        this.UserAddDialog = true
      },
      addUser() {
        this.$refs.UserAddForm.validate(valid => {
          if (valid) {
            console.log('验证成功')
            this.$http.post('/users', this.UserAddForm).then(res => {
              const { meta } = res.data
              if (meta.status === 201) {
                this.UserAddDialog = false
                this.getUserList()
              }
            })
          } else {
            console.log('验证失败')
            return false
          }
        })
      },
      closeUserEditDialog() {
        this.$refs.userEidtForm.resetFields()
      },
      delUserById(id) {
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            this.$http.delete(`users/${id}`).then(res => {
              console.log(res)
              const { meta } = res.data
              if (meta.status === 200) {
                this.$message({
                  type: 'success',
                  message: meta.msg
                })
                this.getUserList()
                this.curPage = 1
              }
              // const index = this.userList.findIndex(item => {
              //   item.id === id
              // })
              // this.userList.splice(index, 1)
              // const totalPage = Math.ceil(this.userList.length / this.pageSize)
              // if (this.currentPage > totalPage) {
              //   this.getUserList(--this.currentPage)
              // }
            })
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
      }
    }
  }
</script>

<style>
  .user-breadcrumb {
    line-height: 30px;
    background-color: #d4dae0;
    font-size: 15px;
    padding-left: 10px;
  }
</style>
