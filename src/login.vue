<template>
  <div class="ui_charcoal login-page application navless">

    <header class="navbar navbar-fixed-top navbar-empty">
      <div class="container">
        <div class="center-logo">
          <img src="./assets/image/logo.png" alt="logo">
        </div>
      </div>
    </header>


    <div class="container navless-container">
      <div class="content">
        <div class="row">
          <div class="col-sm-5 pull-right">
            <div>
              <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="登 陆" name="first"></el-tab-pane>
                <el-tab-pane label="注 册" name="second"></el-tab-pane>
              </el-tabs>
              <div v-show="activeName==='first'" class="login-box">
                <div class="login-heading">
                  <h3>已经有用户? 登陆</h3>
                </div>
                <div class="login-body">
                  <el-form ref="loginForm" :model="loginForm" :rules="registerrule" label-width="80px">
                    <el-form-item label="账 号" prop="account">
                      <el-input placeholder="邮箱" type="text" auto-complete="on" id="username" name="username"
                                v-model="loginForm.account">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="密 码" prop="password">
                      <el-input placeholder="密 码" type="password" auto-complete="on" v-model="loginForm.password">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="验证码" prop="kaptcha">
                      <el-input placeholder="验证码" style="width:80px" v-model="loginForm.kaptcha">
                      </el-input>
                      <img @click="changeKaptcha" :src="kaptchaImg" style="width: 100px">
                    </el-form-item>
                    <el-form-item>
                      <el-button type="primary" :loading="loading" @click="loginSubmit">登 陆</el-button>
                    </el-form-item>
                  </el-form>
                </div>
              </div>
              <div v-show="activeName==='second'" class="login-box">
                <div class="login-heading">
                  <h3>没有用户? 创建新用户</h3>
                </div>
                <div class="login-body">
                  <el-form ref="registerForm" :model="registerForm" :rules="registerrule" label-width="120px">
                    <el-form-item label="姓 名" prop="name">
                      <el-input placeholder="姓 名" v-model="registerForm.name">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="邮 箱" prop="email">
                      <el-input placeholder="邮 箱" v-model="registerForm.email">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="密 码" prop="password">
                      <el-input placeholder="密 码" type="password" v-model="registerForm.password">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="确认密码" prop="password2">
                      <el-input placeholder="确认密码" type="password" v-model="registerForm.password2">
                      </el-input>
                    </el-form-item>
                    <el-form-item label="验证码" prop="kaptcha">
                      <el-input placeholder="验证码" style="width:80px" v-model="registerForm.kaptcha">
                      </el-input>
                      <img @click="changeKaptcha" :src="kaptchaImg" style="width: 100px">
                    </el-form-item>
                    <el-form-item>
                      <el-button type="primary" @click="registerSubmit">注 册</el-button>
                    </el-form-item>
                  </el-form>
                </div>
              </div>
            </div>

          </div>
          <div class="col-sm-7 brand-holder pull-left">
            <h1 class="logoText">
              Go InterFace
            </h1>
            <h3>接口定义神器</h3>
            <p>
              高效开发
            </p>
          </div>
        </div>
      </div>
    </div>
    <hr>
    <div class="container">
      <div class="footer-links">

      </div>
    </div>


  </div>
</template>
<style lang="styl" rel="stylesheet/stylus" scoped type="text/css">
  .logoText
    text-shadow: 0 6px 1px rgba(0, 0, 0, 0.2);
    font-family: 'Lobster', cursive;

  .center-logo
    img
      width 40px
      height 40px

  .login-box
    padding 20px 20px
    border 1px solid #ddd
    border-radius 5px
</style>
<script type="text/ecmascript-6">
  import {mapState} from 'vuex'
  import Server from './extend/Server'
  var SHA256 = require('crypto-js/sha256')

  export default {
    data: function () {
      var validatePass2 = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请再次输入密码'))
        } else if (value !== this.registerForm.password) {
          callback(new Error('两次输入密码不一致!'))
        } else {
          callback()
        }
      }
      return {
        loading: false,
        registerrule: {
          kaptcha: [
            { required: true, message: '输入4位验证码', trigger: 'blur' },
            { min: 4, max: 4, message: '输入4位验证码', trigger: 'blur' }
          ],
          name: [
            { required: true, message: '输入用户名', trigger: 'blur' },
            { min: 2, max: 32, message: '长度在 3 到 32 个字符', trigger: 'blur' }
          ],
          account: [
            { required: true, message: '请输入账号', trigger: 'blur' },
            { min: 6, max: 50, message: '长度在 6 到 50 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '输入密码', trigger: 'blur' },
            { min: 3, max: 15, message: '3-15位大小写字母和._-组成的名称', trigger: 'blur' }
          ],
          password2: [
            { validator: validatePass2, trigger: 'blur' }
          ],
          email: [
            { required: true, message: '请输入邮箱地址', trigger: 'blur' },
            { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur,change' }
          ]
        },
        kaptchaImg: '',
        loginForm: {
          account: '',
          password: '',
          kaptcha: '',
          deskey: '',
          remember: true
        },
        registerForm: {
          email: '',
          kaptcha: '',
          name: '',
          deskey: '',
          password: '',
          password2: ''
        },
        activeName: 'first'
      }
    },
    computed: mapState({
      packageInfo: state => state.app.packageInfo
    }),
    mounted: function () {
      window.EMA.bind('logout', () => {
        this.logout()
      })
      window.EMA.bind('loading.show', () => {
        this.showLoading = true
      })
      window.EMA.bind('loading.hide', () => {
        this.showLoading = false
      })
      window.EMA.bind('alert.show', (title, fn) => {
        this.$alert(title, '注意', {
          callback: action => {
            if (typeof fn == 'function') {
              fn()
            }
          }
        })
      })
      // this.checkLogin()
      this.changeKaptcha()
    },
    methods: {
      checkLogin: function () {
        Server({
          url: 'users/info',
          params: {
            uid: 0
          },
          method: 'get'
        }).then((response) => {
          this.loginSuccess()
        }).catch(() => {
        })
      },
      changeKaptcha: function () {
        Server({
          url: 'kaptcha/init',
          method: 'get'
        }).then((response) => {
          var data = response.data.data
          this.kaptchaImg = 'data:image/jpg;base64,' + data.img
          this.registerForm.deskey = data.key
          this.loginForm.deskey = data.key
        }).catch((e) => {
          this.loading = false
          console.log('err', e)
        })
      },
      parseURL: function (url) {
        var a = document.createElement('a')
        a.href = url
        return {
          params: (function () {
            var ret = {}
            var seg = a.search.replace(/^\?/, '').split('&')
            var len = seg.length
            var i = 0
            var s = null
            for (; i < len; i++) {
              if (!seg[ i ]) {
                continue
              }
              s = seg[ i ].split('=')
              ret[ s[ 0 ] ] = s[ 1 ]
            }
            return ret
          })()
        }
      },
      editPassword: function () {
      },
      handleClick: function (index) {
        this.activeName = index.name
        this.changeKaptcha()
      },
      loginSuccess: function (data) {
        this.loading = false
        // 重定向到退出登陆的页面地址
        // 记录登陆信息。可以是cookie 也可以是 session
        var info = this.parseURL(window.location.href)
        if (info.params.url) {
          window.location.replace(decodeURIComponent(info.params.url))
        } else {
          window.location.replace('index.html')
        }
      },
      loginSubmit () {
        this.$refs.loginForm.validate((valid) => {
          if (valid) {
            this.loading = true
            Server({
              url: 'users/login',
              data: {
                account: this.loginForm.account,
                kaptcha: this.loginForm.kaptcha,
                deskey: this.loginForm.deskey,
                password: SHA256(this.loginForm.password) + ''
              },
              needLoading: true,
              method: 'post'
            }).then((response) => {
              this.loginSuccess(response.data)
              console.log('sucee')
            }).catch((e) => {
              this.loading = false
              console.log('err', e)
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })
        return false
      },
      /**
       * 注册内容提交
       */
      registerSubmit: function () {
        this.$refs.registerForm.validate((valid) => {
          if (valid) {
            this.loading = true
            Server({
              url: 'users/register',
              data: {
                email: this.registerForm.email,
                kaptcha: this.registerForm.kaptcha,
                name: this.registerForm.name,
                deskey: this.registerForm.deskey,
                password: SHA256(this.registerForm.password) + ''
              },
              method: 'post'
            }).then((response) => {
              this.loginSuccess()
            }).catch((e) => {
              this.loading = false
              console.log('err', e)
            })
          } else {
            return false
          }
        })
      },
      resetForm () {
        this.$refs.form.resetFields()
      }
    }
  }
</script>

