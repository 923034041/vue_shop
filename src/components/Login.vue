<template>
  <div class="login_container">
       <div class="logo_box">
           <!-- 登录表单区域 -->
          <el-form  ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="100px">
              <!-- 用户名 -->
               <el-form-item  prop="username" label-width="0px">
                 <el-input  v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
               </el-form-item>
              <!-- 密码 -->
              <el-form-item  prop="password" label-width="0px">
                 <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
               </el-form-item>
               <!-- 按钮区域 -->
               <el-form-item>
                 <el-button type="primary" @click="login">提交</el-button>
                 <el-button type="info"  @click="resetLoginForm">重置</el-button>
               </el-form-item>
             </el-form>
       </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 这是表单的验证规则
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        //   验证密码是否合法
        password: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, max: 6, message: '长度在 3 到 6 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登录失败!')
        this.$message.success('登录成功!')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
  .logo_box {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);
      width: 450px;
      height: 300px;
      background-color: #fff;
      border-radius: 4px;
      box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
     .el-form {
       margin: 80px;
     }
  }
}

</style>
