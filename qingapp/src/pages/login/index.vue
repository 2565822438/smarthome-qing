<template>
  <div>
     <div class="header">
       <div v-if="isLogin">
          <div class="header-title">请登录</div>
          <div class="header-info">Please Login Your Account</div>
       </div>
       <div v-else>
          <div class="header-title">请注册</div>
          <div class="header-info">Please Register Your Account</div>
       </div>
     </div>
     <div class="body">
       <div class="login-form">
         <van-field
            placeholder="请输入用户名"
            :value="inputUserName"
            @change="onUserNameChange"
         />
         <van-field
            type="password"
            placeholder="请输入密码"
            :value="inputPassWord"
            @change="onPassWordChange"
         />
         <div v-if="!isLogin">
            <van-field
            placeholder="请输入找回密码绑定手机号"
            :value="inputContect"
            @change="onContectChange"
         />
         </div>
       </div>
       <van-button slot="button" round block color="#3d7ef9" @click="onClick">{{isLogin ? '登录' : '注册'}}</van-button>
       <div class="other-option">
         <div @click="onOptionClick">
           <span>{{isLogin ? '注册账户' : '登录账户'}}</span>
         </div>
         
         <span style="margin:0 30px">|</span>
         <div @click="onForgetClick">
           <span>忘记密码</span>
         </div>
    
       </div>
       <div class="copyright-wrapper">
         <span class="copyright">Power By Qing</span>
       </div>
       <van-dialog
          use-slot
          title="找回密码校验"
          :show="showFindPW"
          show-cancel-button
          transition = fade
          @confirm="onFindPWConfirm"
          @cancel="onFindPWCancel"
        >
          <van-field
          label = "手机号"
          title = 60px
            placeholder="请输入找回密码绑定手机号"
            :value="inputContect"
            @change="onContectChange"
         />
        </van-dialog>
        <van-dialog
          use-slot
          title="重置密码"
          :show="showResetPW"
          show-cancel-button
          transition = fade
          @confirm="onResetPWConfirm"
          @cancel="onResetPWCancel"
        >
          <van-field
          label = "用户名"
          title = 60px
            :value="inputUserName"
            readonly
         />
         <van-field
          label = "新密码"
          type="password"
          title = 60px
            placeholder="请输入新密码"
            :value="inputPassWord"
            @change="onPassWordChange"
         />
        </van-dialog>
       <van-toast id="van-toast" /> 
     </div>
  </div>
</template>

<script>
import Toast from 'vant-weapp/dist/toast/toast'

export default {
  data () {
    return {
      isLogin: true,
      inputUserName: '',
      inputPassWord: '',
      inputContect: '',
      showFindPW: false,
      showResetPW: false
    }
  },
  methods: {
    onUserNameChange (event) {
      var that = this
      that.$set(that,"inputUserName",event.mp.detail)
      console.log("用户名 = ",event.mp.detail)
    },
    onPassWordChange (event) {
      var that = this
      that.$set(that,"inputPassWord",event.mp.detail)
      console.log("密码 = ",event.mp.detail)
    },
    onContectChange (event) {
      var that = this
      that.$set(that,"inputContect",event.mp.detail)
      console.log("手机号 = ",event.mp.detail)
    },
    onClick (event) {
      var that = this
      Toast.loading({
        duration:0,//持续展示Toast
        forbidClick:true,//禁止背景被点击
        message:that.isLogin?'登陆中...' : '注册中...'
      })
      //模拟请求服务器延时
      setTimeout(()=>{
        if(!that.isLogin){
          wx.setStorage({
            key:"user",
            data:{
              username:that.inputUserName,
              password:that.inputPassWord,
              contect:that.inputContect
            },
            success (res) {
              Toast.success('注册成功')
            },
            fail (res) {
              Toast.success('注册失败')
            }
          })
        }else{
          wx.getStorage({
            key: 'user',
            success (res) {
              if(that.inputUserName === res.data.username &that.inputPassWord === res.data.password) {
                  Toast.success('登陆成功')
                  //500毫秒以后跳转到首页
                  setTimeout(()=>{
                    wx.switchTab({
                      url: '/pages/index/main'
                    })
                  },500)
              } else {
                Toast.fail('用户名或密码错误')
              }
            },
            file (res) {
              console.log(res);
              Toast.fail('数据库暂无注册用户')
            }
          })
        }
      },1000)
    },
    onOptionClick (event) {
      var that = this
      that.isLogin = !that.isLogin
      console.log(`点击注册/登录账户按钮,当前处于${that.isLogin ? '登录' : '注册'}状态`)
    },
    onForgetClick (event) {
      var that = this
     that.showFindPW = true
    },
    onFindPWConfirm (event) {
      var that = this
      wx.getStorage({
        key: 'user',
        success (res) {
          console.log(res.data);
          if(that.inputContect === res.data.contect) {
            that.inputUserName = res.data.username
            that.showResetPW = true
              console.log("找到用户：",res.data.username)
          } else {
            Toast.fail('无该用户')
            that.inputContect = ""
          }
        },
        file (res) {
          console.log(res);
          Toast.fail('数据库暂无注册用户')
          that.inputContect = ""
        }
      })
    },
    onFindPWCancel (event) {
      var that = this
      that.showFindPW = false
      that.inputContect = ""
    },
    onRestctPWConfirm (event) {
      var that = this
      wx.setStorage({
            key:"user",
            data:{
              username:that.inputUserName,
              password:that.inputPassWord,
              contect:that.inputContect
            },
            success (res) {
              console.log(res.data);
              console.log('密码重置成功');
              Toast.success('密码重置成功')
            },
            fail (res) {
              console.log(res.data);
              console.log('密码重置失败');
              Toast.success('密码重置失败')
            }
          })
    },
    onResetPWCancel (event) {
      var that = this
      that.showFindPW = false
      that.inputUserName=""
      that.inputPassWord=""
      that.inputContect=""
    },
  }
}
</script>

<style lang="scss" scoped>
.header {
  height: 100px;
  padding: 25px 0;
  background-color: #3d7ef9;
  color: #fff;
  .header-title{
    font-size: 28px;
    font-weight: 500;
    margin-left: 20px;
  }
  .header-info {
    font-size: 14px;
    margin-left: 20px;
  }
}
.body {
  margin-top: -20px;
  border-radius: 15px 15px 0 0;
  background-color: #fff;
  padding: 20px;
  .login-form {
    margin-bottom: 30px;
  }
  .other-option {
    display: flex;
    justify-content: center;
    margin-top: 20px;
    color: #9f9f9f;
  }
  .copyright-wrapper {
    display: flex;
    justify-content: center;
    .copyright {
      color: #d6d6d6;
      position: fixed;
      bottom: 50px;
    }
  }
}
</style>
