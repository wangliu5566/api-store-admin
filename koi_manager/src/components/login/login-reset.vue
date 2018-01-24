<template>
  <div class="login" :style='{width:winWidth + "px",height:winHeight + "px"}'>
    <div class="content-title login-content-title">
      <div class="content-title-right">
        <i class="el-icon-minus" @click="resizeMin"></i>
        <i class="el-icon-more" @click="resizeMax"></i>
        <i class="el-icon-close" @click="closeForm"></i>
      </div>
    </div>


    <el-row>
      <el-col :span='24'>
        <div class="content">
          <el-form 
          label-position='left' 
          label-width='0px' 
          class='card-box loginform' 
          :model='userInfo' 
          :rules='rule_data' 
          ref='userInfo' 
          v-loading="loading" 
          element-loading-text="系统登录中,请稍后..."
          @keyup.enter.native="loginIn('userInfo')"
          >
            <div class="login-form-title">
              <b>KOI </b> <span>服务端登录</span>
              <!-- <img src="../../../static/images/logo_bak.png" height="40px" width="200px" alt="logo"> -->
            </div>
            <input type="text" style="display:none">
            <input type="password" style="display:none">
            <div style="width:60%;margin:20px auto">
               <el-form-item prop='username'>
                  <el-input type="text" auto-complete="off" autocorrect="off" autocapitalize="off" spellcheck="false" placeholder="用户名/邮箱" v-model='userInfo.username' @change="isChange=true">
                    <template slot="prepend"><i class=""><img src="../../assets/账户.png" alt=""></i></template>
                  </el-input>
                </el-form-item>
                <el-form-item prop='password'>
                  <el-input type="password" auto-complete="off" placeholder="密码" autocorrect="off" autocapitalize="off" spellcheck="false" v-model='userInfo.password' @keyup.enter="loginIn('userInfo')" @change="isChange=true">
                    <template slot="prepend"><i class=""><img src="../../assets/密码.png" alt=""></i></template>
                  </el-input>
                </el-form-item>

                <el-form-item>
                  <el-checkbox v-model="remumber" style="margin-left:20px">记住密码</el-checkbox>
                </el-form-item>

                <el-form-item>
                  <el-button type="primary" class="login-btn" @click="loginIn('userInfo')">登录</el-button>
                </el-form-item>
            </div>
            
          </el-form>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import qs from 'qs';
export default {
  name: 'login',
  data() {
    const CheckLoginPsd = (rule, value, callback) => {
      let psdReg = /^\d{6,12}$/;
      if (value === '') {
        return callback(new Error('密码不能为空！'));
      }else {
        return callback()
      }
    };
    return {
      loading: false,
      winWidth: window.innerWidth,
      winHeight: window.innerHeight,
      userInfo: {
        username: '',
        password: '',
      },

      isChange:false,
      remumber: eval(sessionStorage.getItem('remumber')) || eval(localStorage.getItem('remumber')),
      rule_data: {
        username: [{
          required: true,
          message: '用户名不能为空！',
          trigger: 'blur'
        }],
        password: [{
          validator:CheckLoginPsd,
          required: true,
          trigger: 'blur'
        }],
      }
    }
  },

  methods: {
    setLoginSize() {
      this.winWidth = window.innerWidth ;
      this.winHeight = window.innerHeight ;
    },

    //切换是否记住密码功能
    switchChecked() {
      localStorage.setItem('remumber', !this.remumber);
      sessionStorage.setItem('remumber', !this.remumber);
    },

    //登录系统
    loginIn(userInfo) {

      this.$refs[userInfo].validate((valid) => {
        if (valid) {

          var activePwd = sessionStorage.getItem('password') ? sessionStorage.getItem('password') : (localStorage.getItem('userInfo.password') ? localStorage.getItem('userInfo.password') : '');

          if (this.userInfo.password !== activePwd) {
            this.userInfo.password = hex_sha1(this.userInfo.password);
          }
          this.loading = true;
          this.$http.post('/passport/login?random=' + Math.random(), this.userInfo)
            .then((res) => {
              this.loading = false;
              switch (res.data.Code) {
                 case 200:

                 if(res.data.Data.UserType === 4) {
                       this.$message({
                          message: '您没有权限登录管理后台！',
                          type: 'error'
                        });

                       return;
                   }

                  if (this.remumber == true) {
                    localStorage.setItem('userInfo.password', this.userInfo.password);
                    sessionStorage.setItem('password', this.userInfo.password);
                  } else {
                    localStorage.removeItem('userInfo.password');
                    sessionStorage.removeItem('password');
                  }
                  localStorage.setItem('userInfo.username', this.userInfo.username);
//                localStorage.setItem('userInfo.nickname', res.data.Data.NickName);
                  localStorage.setItem('userInfo.nickname', "张三");
                  localStorage.setItem('userInfo.UserType', res.data.Data.UserType);
                  sessionStorage.setItem('userType',  res.data.Data.UserType);
                  sessionStorage.setItem('nickname',  res.data.Data.RealName);
                  sessionStorage.setItem('username', this.userInfo.username);
                  sessionStorage.setItem('remumber', this.remumber);
                  sessionStorage.setItem('userId', res.data.Data.Id);
                  this.$message({
                    message: '欢迎登录KOI后台管理系统',
                    type: 'success'
                  });
//                sessionStorage.setItem('userInfoIsA',!res.data.Data.IsSupperAdmin);
//                sessionStorage.setItem('userInfoIsS',res.data.Data.IsSupperAdmin);
//                console.log('Hello');
                  this.$router.push({path:'/admin',query:{}});
                  SetWidthHeight(1024,768,true);
                  SetMinSize(1024,768);
                  SetMaxSize(0,0);
//                countRes = 0;
                  break;
                case 34:
                  return;
                  break;
                case 22:
                  if (res.data.Description.indexOf('手机') !== -1) {
                    this.$message({
                      message: res.data.Description,
                      type: 'error',
                      duration: 2000,
                      onClose: () => {
                        this.userInfo.username = ''
                      }
                    })
                  } else if (res.data.Description.indexOf('密码') !== -1) {
                    this.$message({
                      message: res.data.Description,
                      type: 'error',
                      duration: 2000,
                      onClose: () => {
                        this.userInfo.password = ''
                      }
                    })
                  } else if (res.data.Description.indexOf('权限') !== -1) {
                    this.$message({
                      message: res.data.Description,
                      type: 'error',
                      duration: 2000,
                      onClose: () => {
                        this.userInfo.password = ''
                      }
                    })
                  }else if (res.data.Description.indexOf('注册') !== -1) {
                    this.$message({
                      message: res.data.Description,
                      type: 'error',
                      duration: 2000,
                      onClose: () => {
                        this.userInfo.username = '',
                        this.userInfo.password = ''
                      }
                    })
                  }
                  break;
                case 32:
                  this.$confirm('该用户已被锁定，请联系管理员进行解锁！', '提示', {
                    confirmButtonText: '确定',
                    showCancelButton: false,
                    type: 'error'
                  })
                  break;
                case 16:
                  this.$confirm('账号或密码错误！', '提示', {
                    confirmButtonText: '确定',
                    showCancelButton: false,
                    type: 'error'
                  })
                  break;
                case 33:
                  this.$confirm('该用户还未激活，请先联系管理员进行激活！', '提示', {
                    confirmButtonText: '确定',
                    showCancelButton: false,
                    type: 'error'
                  })
                  break;
                default:
                  break;
              }
            })
            .catch((err) => {
              console.log(err)
            })
        } else {
          return false;
        }
      });
    },

    //重置用户名和密码
    reset(userInfo) {
      this.$refs[userInfo].resetFields();
    },
    /**
   * [resizeMin 下面是winform提供的窗口缩放方法]
   * @Author   罗文
   * @DateTime 2017-11-01
   * @return   {[type]}   [description]
   */
    resizeMin() {
      Minimize();
    },
    resizeMax() {
      Maximize();
    },
    closeForm() {
      this.$confirm('确认关闭?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
           Close();
        }).catch(() => {
      
        });
    },
  },
  created() {
    SetWidthHeight(480,400,true);
    SetMinSize(480,400);
    SetMaxSize(480,400);
  },
  mounted() {
    this.setLoginSize();
    //动态监测浏览器窗口变化
    window.onresize = () => {
      this.setLoginSize();
    };
    // hex_sha1(this.userInfo.password)
    //如果sessionStorage或localStorage中存在有值,就直接获取sessionStorage或localStorage中的值
    if (sessionStorage.getItem('username') && sessionStorage.getItem('username') != '') {
      this.userInfo.username = sessionStorage.getItem('username');
      this.userInfo.password = sessionStorage.getItem('password') ? sessionStorage.getItem('password') : (localStorage.getItem('userInfo.password') ? localStorage.getItem('userInfo.password') : '');
    } else {
      this.userInfo.username = localStorage.getItem('userInfo.username') ? localStorage.getItem('userInfo.username') : "";
      if (this.userInfo.username == '') {
        this.userInfo.password = '';
      } else {
        this.userInfo.password = localStorage.getItem('userInfo.password') ? localStorage.getItem('userInfo.password') : "";
      }
    }
  },
  watch: {
    'userInfo.username': function(newVal, oldVal) {
      if (!newVal || newVal.length == 1) {
        this.userInfo.password = '';
      }
    }
  }
}
</script>
<style lang="less">
.el-form-item__error {
  margin-left: 38px;
}

.login {
  background: #e9ecf3;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  .card-box {
    box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);
    -webkit-border-radius: 5px;
    border-radius: 5px;
    -moz-border-radius: 5px;
    background-clip: padding-box;
    margin-bottom: 20px;
    background-color: #fff;
    border: 2px solid #e7ecf1;
    margin-top: 50px;

  }
  .title {
    margin: 10px auto 20px auto;
    text-align: center;
    color: #505458;
    font-family: 'PingFang SC';
  }



  .loginform {
    width: 480px;
    height: 370px;
    /*padding: 35px 35px 20px 35px;*/
  }

  .login-form-title {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100px;
    width: 100%;
    background: #0393e8;
    color:white;
    b {
      font:600 48px '微软雅黑';
    };

    span {
      font:400 28px '微软雅黑';
    }
  }

  .retrieve-password {
    line-height: 46px;
    font-size: 12px;
    color: red;
  }

  .el-input__inner {
    border:none !important; 
    border-bottom: 1px solid #ddd !important; 
  }

  .el-input-group__prepend {
    border:none !important; 
    border-bottom: 1px solid #ddd !important; 
  }
}

.login-content-title {
   height: 35px;
   border-bottom: none;
   box-shadow: 0px 0px 0px #e9ecf3;
   background: #e9ecf3;
   -webkit-app-region: drag;
}

.login-content-title i {
   height: 35px;
   line-height: 35px;
}

.content-title-right {
   display: flex;
   justify-content: center;
   align-items: center;
   height: 40px;
   width: 140px;
   -webkit-app-region: no-drag;
   transition: all ease-in-out 0.3s;
}

// .login-content-title:hover .content-title-right {
//    background: #0393e8;
//    color: white;
// }


.content-title-right i:hover {
  background: #2b579a
}

.login-btn {
  width: 100%;
  border-radius: 20px;
}


</style>
