<template>
  <el-container :style="note">
    <el-form style="height: 360px; background-color: #f7fff2; margin-right: 160px;" :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-position="left" label-width="0px" class="demo-ruleForm login-container">
      <h3 class="title">智慧人力资源管理系统登录</h3>
      <el-form-item prop="account">
        <el-input type="text" v-model="ruleForm2.account" auto-complete="off" placeholder="账号"></el-input>
      </el-form-item>
      <el-form-item prop="checkPass">
        <el-input type="password" v-model="ruleForm2.checkPass" auto-complete="off" placeholder="密码"></el-input>
      </el-form-item>
      <el-checkbox v-model="checked" checked class="remember">记住密码</el-checkbox>
      <el-form-item style="width:100%;">
        <el-button type="primary" style="width:100%;" @click.native.prevent="handleSubmit2" :loading="logining">登录</el-button>
        <!--<el-button @click.native.prevent="handleReset2">重置</el-button>-->
      </el-form-item>
      <el-form-item style="width:100%;">
        <el-button type="info" style="width:100%;" @click.native.prevent="toRegister" :loading="logining">注册</el-button>
      </el-form-item>
    </el-form>
  </el-container>
</template>

<script>
  import { requestLogin } from '../api/api';
  import  defaultImg from '../../static/index.png';
  //import NProgress from 'nprogress'
  export default {
    data() {
      return {
        note:{
            backgroundImage:'url(' + defaultImg + ')',
            backgroundRepeat:'no-repeat',
            backgroundSize: "100% 100%",
            opacity:"0.9",
            height:"98%"

        },

        logining: false,
        ruleForm2: {
          account: 'admin',
          checkPass: '123456'
        },
        rules2: {
          account: [
            { required: true, message: '请输入账号', trigger: 'blur' },
            //{ validator: validaePass }
          ],
          checkPass: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            //{ validator: validaePass2 }
          ]
        },
        checked: true
      };
    },
    methods: {
      toRegister(){
          this.$router.push({ path: '/register' });
      },
      handleReset2() {
        this.$refs.ruleForm2.resetFields();
      },
      handleSubmit2(ev) {
        var _this = this;
        this.$refs.ruleForm2.validate((valid) => {
          if (valid) {
            //_this.$router.replace('/table');
            this.logining = true;
            //NProgress.start();
            var user = { username: this.ruleForm2.account, password: this.ruleForm2.checkPass };
            user.avatar='https://raw.githubusercontent.com/taylorchen709/markdown-images/master/vueadmin/user.png';
            user.name= '张某某';
            // requestLogin(loginParams).then(data => {
            //   this.logining = false;
            //   //NProgress.done();
            //   let { msg, code, user } = data;
            //   if (code !== 200) {
            //     this.$message({
            //       message: msg,
            //       type: 'error'
            //     });
            //   } else {
            //     sessionStorage.setItem('user', JSON.stringify(user));
            //     this.$router.push({ path: '/table' });
            //   }
            // });
              sessionStorage.setItem('user', JSON.stringify(user));
              this.$router.push({ path: '/echarts' });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      }
    }
  }

</script>

<style lang="scss" scoped>
  .login-container {
    /*box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);*/
    -webkit-border-radius: 5px;
    border-radius: 5px;
    -moz-border-radius: 5px;
    background-clip: padding-box;
    margin: 180px auto;
    width: 350px;
    padding: 35px 35px 15px 35px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
    .title {
      margin: 0px auto 40px auto;
      text-align: center;
      color: #505458;
    }
    .remember {
      margin: 0px 0px 35px 0px;
    }
  }


</style>