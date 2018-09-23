<template>
  <div class="loginwrapper">

    <el-row type="flex" class="row-bg loginForm" justify="center" align="middle">
      <el-col :xs="12" :sm="10" :md="8" :lg="6" :xl="4" class="login-content">
        <el-form label-position="top" :model="loginForm" :rules="rules" ref="loginForm" class="demo-loginForm">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="loginForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="loginForm.password"></el-input>
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="submitForm">登录</el-button>
            <el-button @click="resetForm">重置</el-button>
          </el-form-item>
        </el-form>

      </el-col>
    </el-row>
    <!-- <el-button type="primary">主要按钮</el-button> -->

  </div>
</template>

<script>
  // import axios from "axios"
  export default {
    data() {
      return {
        loginForm: {
          username: '',
          password: ''
        },
        rules: {
          username: [
            { required: true, message: '请输入用户名名称', trigger: 'blur' },
            { min: 3, max: 6, message: '长度在 3 到 6个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, max: 6, message: '长度在 3 到 6 个字符', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      login() {
        this.$http
          .post('http://localhost:8888/api/private/v1/login', this.loginForm)
          .then(res => {
            // console.log(res);
            // const data = res.data.data
            // const meta = res.data.meta
            const { data, meta } = res.data
            console.log(data)
            if (meta.status === 200) {
              console.log('登录成功')
              localStorage.setItem('token', data.token)
              this.$router.push('/home')
            } else {
              // this.$message.error("用户名或密码错误");
              this.$message({
                type: 'error',
                message: meta.msg,
                duration: 1000
              })
            }
          })
      },

      submitForm() {
        this.$refs.loginForm.validate(valid => {
          if (valid) {
            this.login()
          } else {
            console.log('error submit!!')
            return false
          }
        })
      },
      resetForm() {
        this.$refs.loginForm.resetFields()
      }
    }
  }
</script>

<style>
  .loginForm,
  .loginwrapper {
    height: 100%;
  }
  .loginForm {
    background-color: skyblue;
  }
  .login-content {
    min-width: 240px;
    padding: 20px 35px;
    border-radius: 10px;
    background-color: #fff;
  }
</style>
