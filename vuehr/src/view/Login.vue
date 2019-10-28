<template>
  <el-form :rules="rules" class="login-container" ref="loginForm" :model="loginForm" v-loading="loading">
    <h3 class="login_title">系统登录</h3>
    <el-form-item prop="username">
      <el-input type="text" v-model="loginForm.username" auto-complete="on" placeholder="账号"></el-input>
    </el-form-item>
    <el-form-item prop="password">
      <el-input type="password" v-model="loginForm.password" auto-complete="off" placeholder="密码"></el-input>
    </el-form-item>
    <el-checkbox class="login_remember" v-model="checked" label-position="left">记住密码</el-checkbox>
    <el-form-item class="login_action">
      <el-button type="primary" @click="submitClick">登录</el-button>
    </el-form-item>
  </el-form>
</template>
<script>
  export default{
    data(){
      return {
        rules: {
          username: [{required: true, message: '请输入用户名', trigger: 'blur'}],
          password: [{required: true, message: '请输入密码', trigger: 'blur'}]
        },
        checked: false,
        loginForm: {
          username: '',
          password: ''
        },
        loading: false
      }
    },
    methods: {
      submitClick () {
        var _this = this;
        _this.$refs['loginForm'].validate((valid) => {
          if (valid) {
            _this.loading = true;
            _this.postRequest('/login', {
              username: _this.loginForm.username,
              password: _this.loginForm.password
            }).then(resp => {
              _this.loading = false
              if (resp && resp.status == 200) {
                var data = resp.data
                _this.$store.commit('login', data.obj)
                var path = _this.$route.query.redirect
                _this.$router.replace({path: path == '/' || path == undefined ? '/home' : path})
              }
              if (_this.checked) {
                _this.setLoginMsg()
              } else {
                _this.removeLoginMsg()
              }
            });
          } else {
            return false
          }
        })
      },
      getLoginMsg () {
        if (this.$cookies.isKey('user_remeber')) {
          var remeber = this.$cookies.get('user_remeber')
          if (remeber === '1') {
            this.checked = true
            this.loginForm.username = this.$cookies.get('user_name')
            this.loginForm.password = this.$cookies.get('user_password')
          } else {
            this.checked = false
          }
        }
      },
      setLoginMsg () {
        this.$cookies.set('user_remeber', 1, 5 * 60)
          .set('user_name', this.loginForm.username, 5 * 60)
          .set('user_password', this.loginForm.password, 5 * 60)
      },
      removeLoginMsg () {
        this.$cookies.remove('user_remeber')
        this.$cookies.remove('user_name')
        this.$cookies.remove('user_password')
      }
    },
    mounted () {
      this.getLoginMsg()
    }
  }
</script>
<style>
  .login-container {
    width: 350px;
    margin: 10% auto;
    border-radius: 15px;
    background: #fff;
    padding: 35px 35px 15px 35px;
    border: 1px solid #eaeaea;
    background-clip: padding-box;
    box-shadow: 0 0 25px #cac6c6;
  }
  .login_title {
    margin: 0px auto 40px auto;
    text-align: center;
    color: #505458;
  }
  .login_remember {
    margin: 0px 0px 35px 0px;
    text-align: left;
  }
  .login_action{
    width: 100%;
  }
  .login_action button{
    width: 100%;
  }
</style>
