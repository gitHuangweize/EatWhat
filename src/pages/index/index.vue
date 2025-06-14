<template>
  <view class="container">
    <!-- 头部区域 -->
    <view class="header">
      <view class="user-info">
        <image class="avatar" :src="userInfo.avatar" mode="aspectFill"></image>
        <view class="greeting">
          <text class="welcome">你好，{{ userInfo.nickname }}</text>
          <text class="today">{{ currentDate }}</text>
        </view>
      </view>
      <view class="weather-info">
        <text class="temperature">{{ weather.temperature }}°C</text>
        <text class="weather-desc">{{ weather.description }}</text>
      </view>
    </view>

    <!-- 今日推荐卡片 -->
    <view class="recommendation-card">
      <view class="card-header">
        <text class="card-title">🎯 今日推荐</text>
        <text class="card-subtitle">根据您的健康状况定制</text>
      </view>
      
      <view class="recommendation-content">
        <view class="main-recommendation">
          <image class="food-image" :src="todayRecommendation.image" mode="aspectFill"></image>
          <view class="food-info">
            <text class="food-name">{{ todayRecommendation.name }}</text>
            <text class="food-benefit">{{ todayRecommendation.benefit }}</text>
            <view class="food-tags">
              <text 
                class="tag" 
                v-for="tag in todayRecommendation.tags" 
                :key="tag"
              >
                {{ tag }}
              </text>
            </view>
          </view>
        </view>
        
        <button class="action-btn" @click="getNewRecommendation">
          <text class="btn-icon">🎲</text>
          <text>换一个推荐</text>
        </button>
      </view>
    </view>

    <!-- 功能卡片区域 -->
    <view class="feature-cards">
      <!-- 健康助手 -->
      <view class="feature-card health-assistant" @click="openHealthAssistant">
        <view class="card-icon">🏥</view>
        <text class="card-title">健康助手</text>
        <text class="card-desc">个性化营养建议</text>
      </view>

      <!-- 季节推荐 -->
      <view class="feature-card seasonal" @click="showSeasonalFoods">
        <view class="card-icon">🍂</view>
        <text class="card-title">当季食材</text>
        <text class="card-desc">{{ currentSeason }}时令蔬果</text>
      </view>

      <!-- 菜谱推荐 -->
      <view class="feature-card recipes" @click="showRecipes">
        <view class="card-icon">📖</view>
        <text class="card-title">菜谱大全</text>
        <text class="card-desc">详细制作步骤</text>
      </view>

      <!-- 营养分析 -->
      <view class="feature-card nutrition" @click="showNutritionAnalysis">
        <view class="card-icon">📊</view>
        <text class="card-title">营养分析</text>
        <text class="card-desc">了解营养成分</text>
      </view>
    </view>

    <!-- 今日食谱 -->
    <view class="daily-recipes">
      <view class="section-header">
        <text class="section-title">今日食谱</text>
        <text class="section-more" @click="showMoreRecipes">更多 ></text>
      </view>
      
      <scroll-view class="recipes-scroll" scroll-x="true" show-scrollbar="false">
        <view class="recipe-list">
          <view 
            class="recipe-item" 
            v-for="recipe in dailyRecipes" 
            :key="recipe.id"
            @click="viewRecipeDetail(recipe)"
          >
            <image class="recipe-image" :src="recipe.image" mode="aspectFill"></image>
            <view class="recipe-info">
              <text class="recipe-name">{{ recipe.name }}</text>
              <view class="recipe-meta">
                <text class="cooking-time">⏱️ {{ recipe.cookingTime }}</text>
                <text class="difficulty">⭐ {{ recipe.difficulty }}</text>
              </view>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 健康小贴士 -->
    <view class="health-tips">
      <view class="section-header">
        <text class="section-title">健康小贴士</text>
      </view>
      
      <view class="tip-card">
        <view class="tip-icon">💡</view>
        <view class="tip-content">
          <text class="tip-title">{{ healthTip.title }}</text>
          <text class="tip-description">{{ healthTip.description }}</text>
        </view>
      </view>
    </view>

    <!-- 快速操作 -->
    <view class="quick-actions">
      <button class="quick-btn" @click="quickSearch">
        <text class="btn-icon">🔍</text>
        <text>快速搜索</text>
      </button>
      
      <button class="quick-btn" @click="randomPick">
        <text class="btn-icon">🎰</text>
        <text>随机选择</text>
      </button>
      
      <button class="quick-btn" @click="addToFavorites">
        <text class="btn-icon">❤️</text>
        <text>添加收藏</text>
      </button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInfo: {
        nickname: '美食爱好者',
        avatar: '/static/default-avatar.png'
      },
      weather: {
        temperature: 24,
        description: '晴朗'
      },
      currentDate: '',
      currentSeason: '秋季',
      todayRecommendation: {
        name: '银耳莲子汤',
        benefit: '滋阴润燥，适合秋季养生',
        image: '/static/foods/yiner-soup.jpg',
        tags: ['润燥', '养颜', '清热']
      },
      dailyRecipes: [
        {
          id: 1,
          name: '番茄鸡蛋面',
          image: '/static/recipes/tomato-noodles.jpg',
          cookingTime: '15分钟',
          difficulty: '简单'
        },
        {
          id: 2,
          name: '蒜蓉西兰花',
          image: '/static/recipes/broccoli.jpg',
          cookingTime: '10分钟',
          difficulty: '简单'
        },
        {
          id: 3,
          name: '红烧肉',
          image: '/static/recipes/braised-pork.jpg',
          cookingTime: '45分钟',
          difficulty: '中等'
        }
      ],
      healthTip: {
        title: '秋季养生重点',
        description: '秋季干燥，应多食用银耳、梨、蜂蜜等润燥食品，少食辛辣，多饮温水。'
      }
    }
  },

  onLoad() {
    this.initPage()
  },

  onShow() {
    this.checkLoginStatus()
  },

  methods: {
    // 初始化页面
    initPage() {
      this.getCurrentDate()
      this.getUserInfo()
      this.getWeatherInfo()
      this.getTodayRecommendation()
    },

    // 检查登录状态
    checkLoginStatus() {
      const token = uni.getStorageSync('token')
      if (!token) {
        uni.reLaunch({
          url: '/pages/login/login'
        })
        return
      }
    },

    // 获取当前日期
    getCurrentDate() {
      const now = new Date()
      const year = now.getFullYear()
      const month = now.getMonth() + 1
      const day = now.getDate()
      const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
      const weekDay = weekDays[now.getDay()]
      
      this.currentDate = `${year}年${month}月${day}日 ${weekDay}`
    },

    // 获取用户信息
    getUserInfo() {
      const userInfo = uni.getStorageSync('userInfo')
      if (userInfo) {
        this.userInfo = userInfo
      }
    },

    // 获取天气信息（模拟）
    getWeatherInfo() {
      // 这里应该调用天气API
      // 模拟数据
      this.weather = {
        temperature: Math.floor(Math.random() * 20) + 15,
        description: ['晴朗', '多云', '阴天'][Math.floor(Math.random() * 3)]
      }
    },

    // 获取今日推荐
    getTodayRecommendation() {
      // 这里应该根据用户画像和健康状况调用推荐算法
      const recommendations = [
        {
          name: '银耳莲子汤',
          benefit: '滋阴润燥，适合秋季养生',
          image: '/static/foods/yiner-soup.jpg',
          tags: ['润燥', '养颜', '清热']
        },
        {
          name: '山药排骨汤',
          benefit: '健脾养胃，增强免疫力',
          image: '/static/foods/yam-soup.jpg',
          tags: ['健脾', '养胃', '营养']
        },
        {
          name: '菠菜鸡蛋汤',
          benefit: '补铁补血，营养丰富',
          image: '/static/foods/spinach-soup.jpg',
          tags: ['补血', '营养', '清淡']
        }
      ]
      
      this.todayRecommendation = recommendations[Math.floor(Math.random() * recommendations.length)]
    },

    // 获取新推荐
    getNewRecommendation() {
      uni.showLoading({
        title: '获取推荐中...'
      })
      
      setTimeout(() => {
        this.getTodayRecommendation()
        uni.hideLoading()
        
        uni.showToast({
          title: '已为您推荐新食谱',
          icon: 'success'
        })
      }, 1000)
    },

    // 打开健康助手
    openHealthAssistant() {
      uni.showModal({
        title: '健康助手',
        content: '请告诉我您的健康状况（如孕妇、糖尿病、高血压等），我将为您提供个性化的饮食建议。',
        confirmText: '设置',
        success: (res) => {
          if (res.confirm) {
            // 跳转到健康设置页面
            uni.showToast({
              title: '功能开发中',
              icon: 'none'
            })
          }
        }
      })
    },

    // 显示当季食材
    showSeasonalFoods() {
      const seasonalFoods = {
        '春季': ['春笋', '韭菜', '菠菜', '荠菜'],
        '夏季': ['西瓜', '黄瓜', '番茄', '茄子'],
        '秋季': ['南瓜', '红薯', '栗子', '柿子'],
        '冬季': ['白萝卜', '白菜', '红枣', '桂圆']
      }
      
      const foods = seasonalFoods[this.currentSeason] || []
      
      uni.showModal({
        title: `${this.currentSeason}时令食材`,
        content: `推荐食材：${foods.join('、')}`,
        showCancel: false
      })
    },

    // 显示菜谱
    showRecipes() {
      uni.showToast({
        title: '菜谱功能开发中',
        icon: 'none'
      })
    },

    // 显示营养分析
    showNutritionAnalysis() {
      uni.showToast({
        title: '营养分析功能开发中',
        icon: 'none'
      })
    },

    // 显示更多食谱
    showMoreRecipes() {
      uni.showToast({
        title: '更多食谱功能开发中',
        icon: 'none'
      })
    },

    // 查看食谱详情
    viewRecipeDetail(recipe) {
      uni.showModal({
        title: recipe.name,
        content: `烹饪时间：${recipe.cookingTime}\n难度：${recipe.difficulty}\n\n详细制作步骤功能开发中...`,
        showCancel: false
      })
    },

    // 快速搜索
    quickSearch() {
      uni.showToast({
        title: '搜索功能开发中',
        icon: 'none'
      })
    },

    // 随机选择
    randomPick() {
      uni.showLoading({
        title: '随机选择中...'
      })
      
      setTimeout(() => {
        uni.hideLoading()
        this.getNewRecommendation()
      }, 1500)
    },

    // 添加到收藏
    addToFavorites() {
      uni.showToast({
        title: '已添加到收藏',
        icon: 'success'
      })
    }
  }
}
</script>

<style scoped>
.container {
  background: #F8F9FA;
  min-height: 100vh;
  padding-bottom: 120rpx;
}

/* 头部区域 */
.header {
  background: linear-gradient(135deg, #FF6B6B 0%, #FF8E8E 100%);
  padding: 40rpx 30rpx 30rpx;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.user-info {
  display: flex;
  align-items: center;
}

.avatar {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  margin-right: 20rpx;
}

.greeting {
  display: flex;
  flex-direction: column;
}

.welcome {
  font-size: 32rpx;
  font-weight: 600;
  color: white;
  margin-bottom: 8rpx;
}

.today {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.8);
}

.weather-info {
  text-align: right;
}

.temperature {
  display: block;
  font-size: 36rpx;
  font-weight: 600;
  color: white;
  margin-bottom: 4rpx;
}

.weather-desc {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.8);
}

/* 推荐卡片 */
.recommendation-card {
  background: white;
  margin: 30rpx;
  border-radius: 20rpx;
  padding: 30rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.08);
}

.card-header {
  margin-bottom: 30rpx;
}

.card-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.card-subtitle {
  font-size: 24rpx;
  color: #999;
}

.main-recommendation {
  display: flex;
  margin-bottom: 30rpx;
}

.food-image {
  width: 120rpx;
  height: 120rpx;
  border-radius: 12rpx;
  margin-right: 20rpx;
}

.food-info {
  flex: 1;
}

.food-name {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.food-benefit {
  font-size: 26rpx;
  color: #666;
  display: block;
  margin-bottom: 12rpx;
}

.food-tags {
  display: flex;
  flex-wrap: wrap;
}

.tag {
  background: #FFF0F0;
  color: #FF6B6B;
  font-size: 20rpx;
  padding: 4rpx 12rpx;
  border-radius: 12rpx;
  margin-right: 12rpx;
  margin-bottom: 8rpx;
}

.action-btn {
  width: 100%;
  height: 80rpx;
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 12rpx;
  font-size: 28rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-icon {
  margin-right: 8rpx;
}

/* 功能卡片 */
.feature-cards {
  padding: 0 30rpx;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20rpx;
  margin-bottom: 40rpx;
}

.feature-card {
  background: white;
  padding: 30rpx;
  border-radius: 16rpx;
  text-align: center;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.card-icon {
  font-size: 48rpx;
  margin-bottom: 16rpx;
}

.feature-card .card-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.card-desc {
  font-size: 22rpx;
  color: #999;
}

/* 今日食谱 */
.daily-recipes {
  margin-bottom: 40rpx;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 30rpx;
  margin-bottom: 20rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
}

.section-more {
  font-size: 24rpx;
  color: #FF6B6B;
}

.recipes-scroll {
  white-space: nowrap;
}

.recipe-list {
  display: flex;
  padding-left: 30rpx;
}

.recipe-item {
  display: inline-block;
  width: 240rpx;
  background: white;
  border-radius: 16rpx;
  margin-right: 20rpx;
  overflow: hidden;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.recipe-image {
  width: 100%;
  height: 160rpx;
}

.recipe-info {
  padding: 20rpx;
}

.recipe-name {
  font-size: 26rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 12rpx;
}

.recipe-meta {
  display: flex;
  justify-content: space-between;
}

.cooking-time,
.difficulty {
  font-size: 20rpx;
  color: #999;
}

/* 健康小贴士 */
.health-tips {
  padding: 0 30rpx;
  margin-bottom: 40rpx;
}

.tip-card {
  background: white;
  padding: 30rpx;
  border-radius: 16rpx;
  display: flex;
  align-items: flex-start;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.tip-icon {
  font-size: 40rpx;
  margin-right: 20rpx;
}

.tip-content {
  flex: 1;
}

.tip-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 12rpx;
}

.tip-description {
  font-size: 24rpx;
  color: #666;
  line-height: 1.6;
}

/* 快速操作 */
.quick-actions {
  display: flex;
  padding: 0 30rpx;
  gap: 20rpx;
}

.quick-btn {
  flex: 1;
  height: 80rpx;
  background: white;
  border: 2rpx solid #FF6B6B;
  color: #FF6B6B;
  border-radius: 12rpx;
  font-size: 24rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.quick-btn .btn-icon {
  margin-right: 8rpx;
}
</style>
