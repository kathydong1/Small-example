<template>
  <div class="login_vip">
    <h3 class="login">注册会员</h3>
    <div class="options">
      <p>手机号</p>
      <input type="number" placeholder="请输入您的手机号" maxlength="11" v-model="authTel">
    </div>
    <div class="password">
      <p>验证码</p>
      <input type="number" placeholder="请输入验证码" v-model="authVal" maxlength="4">
      <div class="sms_click" @click="sendSms">{{countdownInfo}}</div>
    </div>
    <button class="btn" plain="true" hover-class="none" style="color:#fff;" @click="commitSms">下一步</button>
  </div>
</template>

<script>
  import gcoord from 'gcoord';
  export default {
    data() {
      return {
        authTel: '',
        authVal: '',
        countdown: null,
        countdownInfo: '获取验证码',
        countdownTimer: null,
        isSend: true,
      }
    },
    onReady() {
      clearInterval(this.countdownTimer)
      this.countdown = null;
      this.countdownInfo = '获取验证码';
      this.countdownTimer = null;
      this.isSend = true;
      this.authTel = wx.getStorageSync('loginInfo') ? wx.getStorageSync('loginInfo').Phone : '';
      this.authVal = '';
    },
    methods: {
      //发送验证码
      sendSms() {
        // console.log(this.authTel)
        if (this.phone(this.authTel)) {
          if (this.isSend) {
            this.isSend = false;
            this.util.post({
              url: '/api/Customer/VerifyCode/SendSmsCode',
              data: {
                Mobile: this.authTel,
                BizType: 8
              }
            }).then(res => {
              this.msg(res.Msg)
              this.countdown = 60;
              this.countdownInfo = `${this.countdown}s后重新获取`;
              this.countdownTimer = setInterval(() => {
                this.countdown--;
                this.countdownInfo = `${this.countdown}s后重新获取`;
                if (this.countdown <= 0) {
                  clearInterval(this.countdownTimer)
                  this.countdownInfo = '重新获取';
                  this.isSend = true;
                }
              }, 1000)
            }).catch(err => {
              this.isSend = true;
              this.msg(err.Msg)
            })
          }
        }
      },
      commitSms(userInfo) {
        if (this.phone(this.authTel) && this.smsCoding(this.authVal)) {
          let QQmap = wx.getStorageSync('QQmap');
          var result = gcoord.transform(
            [QQmap.longitude, QQmap.latitude], // 经纬度坐标
            gcoord.WGS84, // 当前坐标系
            gcoord.BD09 // 目标坐标系
          );
          // console.log(result)
          this.util.post({
            url: '/api/Customer/VerifyCode/CommitSmsCode',
            data: {
              Mobile: this.authTel,
              BizType: 8,
              VerifyCode: this.authVal,
              Loction: `${result[0]},${result[1]}`,
              CityName: QQmap.city,
              wxUserInfo: ''
            }
          }).then(res => {
            // console.log(res)
            //登录成功清除定时器
            this.timer = null;
            clearInterval(this.countdownTimer)
            //存储信息
            let vipObj = {
              token: res.Body.Token,
              tel: this.authTel
            }
            wx.setStorageSync('InpVip', vipObj)
            //跳转至信息填写页
            wx.redirectTo({
              url: '/pages/inp-info/main'
            })
          }).catch(err => {
            this.msg(err.Msg)
          })
        }
      },
      //检测手机号
      phone(tel) {
        let reg = /^[1][3,4,5,6,7,8,9]\d{9}$/;
        if (reg.test(tel)) {
          return true;
        } else {
          if (tel != '') {
            this.msg('请输入正确的手机号')
          } else {
            this.msg('请输入手机号')
          }
          return false;
        }
      },
      smsCoding(val) { //短信4位
        let reg = /^\d{4}$/;
        if (reg.test(val)) {
          return true;
        } else {
          if (val != '') {
            this.msg('请输入完整的短信验证码')
          } else {
            this.msg('请输入短信验证码')
          }
          return false;
        }
      },
    },
    watch: {
      authTel: function(newVal, oldVal) {
        this.authTel = newVal.replace(/[^\d]/g, '');
      },
      authVal: function(newVal, oldVal) {
        this.authVal = newVal.replace(/[^\d]/g, '');
      },
    },
    onUnload() {
      clearInterval(this.countdownTimer)
      this.countdownTimer = null;
    }
  }
</script>

<style lang="less">
  .login_vip {
    height: 100%;
    overflow-x: hidden;
    position: relative;
    .login {
      margin: 20rpx 36rpx;
      font-size: 46rpx;
      font-weight: 700;
    }
    .options {
      background: #fff;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      padding: 20rpx 0;
      margin: 0 36rpx;
      position: relative;
      p {
        font-size: 28rpx;
        color: #333;
        margin-right: 38rpx;
        transform: translateY(3rpx);
      }
      input {
        flex: 1;
        font-size: 28rpx;
        color: #666;
        height: 60rpx;
      }
      &:after {
        content: '';
        display: block;
        width: 100%;
        height: 0;
        border-bottom: 1px solid #ebebeb;
        position: absolute;
        bottom: 0;
        left: 0;
        transform: scaleY(0.5);
        transform-origin: 0 0;
      }
    }
    .password {
      background: #fff;
      padding: 20rpx 0;
      margin: 0 36rpx;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      position: relative;
      p {
        font-size: 28rpx;
        color: #333;
        margin-right: 38rpx;
        transform: translateY(3rpx);
      }
      input {
        flex: 1;
        font-size: 28rpx;
        color: #666;
        padding-right: 20rpx;
        height: 60rpx;
      }
      .sms_click {
        height: 50rpx;
        line-height: 50rpx;
        font-size: 24rpx;
        color: #ff4d3a;
        background: transparent;
        border: 1rpx solid #ff4d3a;
        border-radius: 6rpx;
        outline: none;
        white-space: nowrap;
        padding: 0 10rpx;
      }
      &:after {
        content: '';
        display: block;
        width: 100%;
        height: 0;
        border-bottom: 1px solid #ebebeb;
        position: absolute;
        bottom: 0;
        left: 0;
        transform: scaleY(0.5);
        transform-origin: 0 0;
      }
    }
    .btn {
      background: #333;
      margin: 50rpx 36rpx 20rpx;
      text-align: center;
      height: 88rpx;
      line-height: 88rpx;
      border-radius: 6rpx;
      color: #fff;
      font-size: 30rpx;
    }
  }
</style>
