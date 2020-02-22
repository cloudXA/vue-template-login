<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" auto-complete="on" label-position="left">
      <div class="title-container">
        <h3 class="title">登录</h3>
      </div>

      <el-form-item prop="username">
        <span class="svg-container">
          <svg-icon icon-class="user"></svg-icon>
        </span>
        <el-input 
            ref="username"
            v-model="loginForm.username"
            placeholder="用户名"
            name="username"
            type="text"
            tabindex="50"
            auto-complete="on" 
        />
      </el-form-item>

      <el-form-item prop="password">
        <span class="svg-container">
          <svg-icon icon-class="password"></svg-icon>
        </span>
        <el-input 
            :key="passwordType"
            ref="password"
            v-model="loginForm.password"
            placeholder="密码"
            name="password"
            :type="passwordType"
            tabindex="50"
            auto-complete="on"
            @keyup.enter.native = 'handleLogin'
        />
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>
      <el-button :loading="loading" type="primary" :style="styleButton" @click.native.prevent="handleLogin">登录</el-button>
    </el-form>
  </div>
</template>

<script>
import { validUsername } from '@/utils/validate'

export default {
  name: 'Login',
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!validUsername(value)) {
        callback(new Error('请输入正确的名字'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6 ) {
        callback(new Error('密码个数不能少于6为'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: 'admin',
        password: ''
      },
      // 前端页面级表单校验
      loginRules: { // loginRules为el-form中的:rules="loginRules"的绑定属性
        username: [{ required: true, trigger: 'blur', validator: validateUsername}], // username 为el-form-item的prop的属性值; validateUsername为回调函数，接收rule，value,callback. 主要判断value是否符合输入规范
        password: [{ required: true, trigger: 'blur', validator: validatePassword}]
      },
      styleButton: {
        width: '100%',
        marginBottom: '30px'
      },
      passwordType: 'password',
      redirect: undefined,
      loading: false
    }
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = 'text'
      } else {
        this.passwordType = 'password'
      }
      // 上述语句执行后执行下面 
      /**
       * @description 返回一个promise，返回后输入框聚焦
       */
      this.$nextTick().then(() => this.$refs.password.focus())
      // this.$nextTick(() => {
      //   this.$refs.password.focus()
      // })
    },
    // 向后端请求数据校验 
    handleLogin() {
      this.$refs.loginForm.validate(valid => { //判断全部表单验证是否通过，通过为true，不通过为false
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || '/' })
            this.loading = false 
          })
        } else {
          return false
        }
      })
    }
  },
  // 开始进入'/'然后跳转到'/login',路由对象变化，触发handler函数，监听$route对象对象的变化，变化后执行内的handler
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect
        alert(this.redirect)
      },
      immediate: true
    }
  }
}

</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */


$cursor: rgb(255, 0, 0);

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

// el-form {
//   el-input {
//     width: 50px;

//   }
// }


</style>


