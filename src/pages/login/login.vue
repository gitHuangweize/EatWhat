<template>
  <view class="login-container">
    <!-- 自定义导航栏 -->
    <view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
      <view class="navbar-content">
        <text class="navbar-title">登录</text>
      </view>
    </view>
    
    <!-- 主要内容 -->
    <view class="content">
      <!-- Logo区域 -->
      <view class="logo-section">
        <view class="logo">🍽️</view>
        <text class="app-name">今天吃什么</text>
        <text class="app-slogan">让每一餐都有意义</text>
      </view>
      
      <!-- 登录表单 -->
      <view class="login-form">
        <view class="input-group">
          <view class="input-item">
            <text class="input-icon">📱</text>
            <input 
              class="input" 
              type="number" 
              placeholder="请输入手机号" 
              v-model="phoneNumber"
              maxlength="11"
            />
          </view>
          
          <view class="input-item">
            <text class="input-icon">🔐</text>
            <input 
              class="input" 
              type="number" 
              placeholder="请输入验证码" 
              v-model="verifyCode"
              maxlength="6"
            />
            <button 
              class="verify-btn" 
              :disabled="!canSendCode || counting"
              @click="sendVerifyCode"
            >
              {{ counting ? `${countdown}s后重发` : '发送验证码' }}
            </button>
          </view>
        </view>
        
        <button class="login-btn" @click="handleLogin" :disabled="!canLogin">
          登录
        </button>
      </view>
      
      <!-- 其他登录方式 -->
      <view class="other-login">
        <view class="divider">
          <text class="divider-text">其他登录方式</text>
        </view>
        
        <view class="social-login">
          <button class="social-btn wechat" @click="wechatLogin">
            <text class="social-icon">💬</text>
            <text>微信登录</text>
          </button>
        </view>
      </view>
      
      <!-- 协议同意 -->
      <view class="agreement">
        <checkbox-group @change="checkboxChange">
          <checkbox class="agreement-checkbox" :checked="agreedToTerms" />
        </checkbox-group>
        <text class="agreement-text">
          我已阅读并同意
          <text class="link" @click="showPrivacy">《隐私政策》</text>
          和
          <text class="link" @click="showTerms">《服务条款》</text>
        </text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      statusBarHeight: 0, // 状态栏高度
      phoneNumber: '',
      verifyCode: '',
      countdown: 60,
      counting: false,
      agreedToTerms: false
    }
  },
  
  computed: {
    // 是否可以发送验证码
    canSendCode() {
      return this.phoneNumber.length === 11 && /^1[3-9]\d{9}$/.test(this.phoneNumber)
    },
    
    // 是否可以登录
    canLogin() {
      return this.canSendCode && 
             this.verifyCode.length === 6 && 
             this.agreedToTerms
    }
  },
  
  onLoad() {
    // 获取状态栏高度
    this.getSystemInfo()
    
    // 检查是否已登录
    this.checkLoginStatus()
  },
  
  methods: {
    // 获取系统信息
    getSystemInfo() {
      const systemInfo = uni.getSystemInfoSync()
      this.statusBarHeight = systemInfo.statusBarHeight || 20
    },
    
    // 检查登录状态
    checkLoginStatus() {
      const token = uni.getStorageSync('token')
      if (token) {
        // 已登录，跳转到首页
        uni.reLaunch({
          url: '/pages/index/index'
        })
      }
    },
    
    // 发送验证码
    sendVerifyCode() {
      if (!this.canSendCode) return
      
      // 这里应该调用后端接口发送验证码
      uni.showToast({
        title: '验证码已发送',
        icon: 'success'
      })
      
      // 开始倒计时
      this.startCountdown()
    },
    
    // 开始倒计时
    startCountdown() {
      this.counting = true
      this.countdown = 60
      
      const timer = setInterval(() => {
        this.countdown--
        
        if (this.countdown <= 0) {
          clearInterval(timer)
          this.counting = false
          this.countdown = 60
        }
      }, 1000)
    },
    
    // 处理登录
    async handleLogin() {
      if (!this.canLogin) return
      
      try {
        uni.showLoading({
          title: '登录中...'
        })
        
        // 这里应该调用后端登录接口
        // const result = await this.loginApi()
        
        // 模拟登录成功
        setTimeout(() => {
          uni.hideLoading()
          
          // 保存登录信息
          uni.setStorageSync('token', 'demo_token_123')
          uni.setStorageSync('userInfo', {
            phone: this.phoneNumber,
            nickname: '美食爱好者',
            avatar: '/static/default-avatar.png'
          })
          
          uni.showToast({
            title: '登录成功',
            icon: 'success'
          })
          
          // 跳转到首页
          setTimeout(() => {
            uni.reLaunch({
              url: '/pages/index/index'
            })
          }, 1500)
        }, 2000)
        
      } catch (error) {
        uni.hideLoading()
        uni.showToast({
          title: '登录失败',
          icon: 'none'
        })
      }
    },
    
    // 微信登录
    wechatLogin() {
      // #ifdef MP-WEIXIN
      uni.getUserProfile({
        desc: '用于完善会员资料',
        success: (res) => {
          console.log('微信用户信息', res.userInfo)
          // 处理微信登录逻辑
        }
      })
      // #endif
      
      // #ifdef H5
      uni.showToast({
        title: '微信登录功能开发中',
        icon: 'none'
      })
      // #endif
    },
    
    // 复选框改变
    checkboxChange(e) {
      this.agreedToTerms = e.detail.value.length > 0
    },
    
    // 显示隐私政策
    showPrivacy() {
      uni.showModal({
        title: '隐私政策',
        content: '这里是隐私政策的内容...',
        showCancel: false
      })
    },
    
    // 显示服务条款
    showTerms() {
      uni.showModal({
        title: '服务条款',
        content: '这里是服务条款的内容...',
        showCancel: false
      })
    }
  }
}
</script>

<style scoped>
.login-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #FF6B6B 0%, #FF8E8E 100%);
}

.custom-navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background: transparent;
  z-index: 999;
}

.navbar-content {
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.navbar-title {
  font-size: 18px;
  font-weight: 600;
  color: white;
}

.content {
  padding-top: 88px;
  padding: 88px 40rpx 40rpx;
}

.logo-section {
  text-align: center;
  margin-bottom: 80rpx;
}

.logo {
  font-size: 120rpx;
  margin-bottom: 20rpx;
}

.app-name {
  display: block;
  font-size: 48rpx;
  font-weight: bold;
  color: white;
  margin-bottom: 10rpx;
}

.app-slogan {
  display: block;
  font-size: 28rpx;
  color: rgba(255, 255, 255, 0.8);
}

.login-form {
  background: white;
  border-radius: 20rpx;
  padding: 60rpx 40rpx;
  margin-bottom: 60rpx;
  box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 60rpx;
}

.input-item {
  display: flex;
  align-items: center;
  background: #F8F9FA;
  border-radius: 12rpx;
  padding: 0 30rpx;
  margin-bottom: 30rpx;
  height: 100rpx;
}

.input-icon {
  font-size: 32rpx;
  margin-right: 20rpx;
}

.input {
  flex: 1;
  height: 100rpx;
  font-size: 30rpx;
  border: none;
  outline: none;
}

.verify-btn {
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 8rpx;
  padding: 0 20rpx;
  height: 60rpx;
  font-size: 24rpx;
  line-height: 60rpx;
}

.verify-btn:disabled {
  background: #ccc;
}

.login-btn {
  width: 100%;
  height: 100rpx;
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 12rpx;
  font-size: 32rpx;
  font-weight: 600;
}

.login-btn:disabled {
  background: #ccc;
}

.other-login {
  margin-bottom: 60rpx;
}

.divider {
  text-align: center;
  margin-bottom: 40rpx;
}

.divider-text {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.7);
  background: linear-gradient(135deg, #FF6B6B 0%, #FF8E8E 100%);
  padding: 0 20rpx;
}

.social-login {
  display: flex;
  justify-content: center;
}

.social-btn {
  display: flex;
  align-items: center;
  background: white;
  border: none;
  border-radius: 12rpx;
  padding: 20rpx 40rpx;
  font-size: 28rpx;
  color: #333;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.1);
}

.social-icon {
  font-size: 32rpx;
  margin-right: 10rpx;
}

.agreement {
  display: flex;
  align-items: flex-start;
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.8);
}

.agreement-checkbox {
  margin-right: 10rpx;
  transform: scale(0.8);
}

.agreement-text {
  flex: 1;
  line-height: 1.5;
}

.link {
  color: white;
  text-decoration: underline;
}
</style> 