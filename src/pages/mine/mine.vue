<template>
  <view class="container">
    <!-- 个人信息卡片 -->
    <view class="profile-card">
      <view class="profile-header">
        <image class="avatar-large" :src="userInfo.avatar" mode="aspectFill"></image>
        <view class="user-details">
          <text class="username">{{ userInfo.nickname }}</text>
          <text class="user-desc">{{ userInfo.description || '还没有个人介绍' }}</text>
          <button class="edit-btn" size="mini" @click="editProfile">编辑资料</button>
        </view>
      </view>
      
      <view class="stats-row">
        <view class="stat-item" @click="viewFavorites">
          <text class="stat-number">{{ stats.favorites }}</text>
          <text class="stat-label">收藏</text>
        </view>
        <view class="stat-item" @click="viewRecords">
          <text class="stat-number">{{ stats.records }}</text>
          <text class="stat-label">记录</text>
        </view>
        <view class="stat-item" @click="viewFollowing">
          <text class="stat-number">{{ stats.following }}</text>
          <text class="stat-label">关注</text>
        </view>
      </view>
    </view>

    <!-- 我的记录 -->
    <view class="section">
      <view class="section-header">
        <text class="section-title">我的记录</text>
        <text class="section-more" @click="addNewRecord">+ 添加</text>
      </view>
      
      <view class="records-list">
        <view 
          class="record-item" 
          v-for="record in myRecords" 
          :key="record.id"
          @click="viewRecordDetail(record)"
        >
          <image class="record-image" :src="record.image" mode="aspectFill"></image>
          <view class="record-info">
            <text class="record-name">{{ record.name }}</text>
            <text class="record-location">{{ record.location }}</text>
            <view class="record-meta">
              <text class="record-date">{{ record.date }}</text>
              <view class="record-rating">
                <text class="star" v-for="n in 5" :key="n" :class="{ active: n <= record.rating }">⭐</text>
              </view>
            </view>
          </view>
          <view class="record-category">
            <text class="category-tag" :class="record.category">{{ getCategoryName(record.category) }}</text>
          </view>
        </view>
      </view>
    </view>

    <!-- 分类管理 -->
    <view class="section">
      <view class="section-header">
        <text class="section-title">分类管理</text>
      </view>
      
      <view class="category-grid">
        <view 
          class="category-item" 
          v-for="category in categories" 
          :key="category.key"
          @click="viewCategoryRecords(category)"
        >
          <view class="category-icon">{{ category.icon }}</view>
          <text class="category-name">{{ category.name }}</text>
          <text class="category-count">{{ category.count }}条记录</text>
        </view>
      </view>
    </view>

    <!-- 功能菜单 -->
    <view class="section">
      <view class="menu-list">
        <view class="menu-item" @click="viewHealthProfile">
          <view class="menu-icon">🏥</view>
          <text class="menu-text">健康档案</text>
          <text class="menu-arrow">></text>
        </view>
        
        <view class="menu-item" @click="viewSettings">
          <view class="menu-icon">⚙️</view>
          <text class="menu-text">设置</text>
          <text class="menu-arrow">></text>
        </view>
        
        <view class="menu-item" @click="viewHelp">
          <view class="menu-icon">❓</view>
          <text class="menu-text">帮助与反馈</text>
          <text class="menu-arrow">></text>
        </view>
        
        <view class="menu-item" @click="viewAbout">
          <view class="menu-icon">ℹ️</view>
          <text class="menu-text">关于我们</text>
          <text class="menu-arrow">></text>
        </view>
      </view>
    </view>

    <!-- 退出登录 -->
    <view class="logout-section">
      <button class="logout-btn" @click="logout">退出登录</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInfo: {
        nickname: '美食爱好者',
        avatar: '/static/default-avatar.png',
        description: '热爱美食，享受生活'
      },
      stats: {
        favorites: 23,
        records: 15,
        following: 8
      },
      myRecords: [
        {
          id: 1,
          name: '川菜馆的麻婆豆腐',
          location: '蜀香园川菜馆',
          image: '/static/foods/mapo-tofu.jpg',
          date: '2024-01-15',
          rating: 5,
          category: 'special'
        },
        {
          id: 2,
          name: '妈妈的家常菜',
          location: '家里',
          image: '/static/foods/home-cooking.jpg',
          date: '2024-01-12',
          rating: 5,
          category: 'home'
        },
        {
          id: 3,
          name: '下午茶时光',
          location: '星巴克',
          image: '/static/foods/afternoon-tea.jpg',
          date: '2024-01-10',
          rating: 4,
          category: 'tea'
        }
      ],
      categories: [
        {
          key: 'special',
          name: '特色菜',
          icon: '🍽️',
          count: 8
        },
        {
          key: 'home',
          name: '家常菜',
          icon: '🏠',
          count: 12
        },
        {
          key: 'tea',
          name: '喝茶类',
          icon: '🍵',
          count: 5
        },
        {
          key: 'hotpot',
          name: '火锅烤肉',
          icon: '🔥',
          count: 3
        },
        {
          key: 'expensive',
          name: '奢侈',
          icon: '💎',
          count: 2
        },
        {
          key: 'affordable',
          name: '实惠',
          icon: '💰',
          count: 15
        }
      ]
    }
  },

  onLoad() {
    this.loadUserData()
  },

  methods: {
    // 加载用户数据
    loadUserData() {
      const userInfo = uni.getStorageSync('userInfo')
      if (userInfo) {
        this.userInfo = { ...this.userInfo, ...userInfo }
      }
    },

    // 获取分类名称
    getCategoryName(categoryKey) {
      const category = this.categories.find(cat => cat.key === categoryKey)
      return category ? category.name : '未分类'
    },

    // 编辑个人资料
    editProfile() {
      uni.showModal({
        title: '编辑资料',
        content: '个人资料编辑功能开发中...',
        showCancel: false
      })
    },

    // 查看收藏
    viewFavorites() {
      uni.showToast({
        title: '收藏功能开发中',
        icon: 'none'
      })
    },

    // 查看记录
    viewRecords() {
      uni.showToast({
        title: '记录功能开发中',
        icon: 'none'
      })
    },

    // 查看关注
    viewFollowing() {
      uni.showToast({
        title: '关注功能开发中',
        icon: 'none'
      })
    },

    // 添加新记录
    addNewRecord() {
      uni.showActionSheet({
        itemList: ['拍照记录', '从相册选择', '手动添加'],
        success: (res) => {
          const actions = [
            () => this.takePhoto(),
            () => this.chooseFromAlbum(),
            () => this.manualAdd()
          ]
          
          if (actions[res.tapIndex]) {
            actions[res.tapIndex]()
          }
        }
      })
    },

    // 拍照记录
    takePhoto() {
      uni.chooseImage({
        count: 1,
        sourceType: ['camera'],
        success: (res) => {
          // 处理拍照结果
          console.log('拍照结果:', res)
          uni.showToast({
            title: '拍照记录功能开发中',
            icon: 'none'
          })
        }
      })
    },

    // 从相册选择
    chooseFromAlbum() {
      uni.chooseImage({
        count: 1,
        sourceType: ['album'],
        success: (res) => {
          // 处理选择结果
          console.log('相册选择结果:', res)
          uni.showToast({
            title: '相册选择功能开发中',
            icon: 'none'
          })
        }
      })
    },

    // 手动添加
    manualAdd() {
      uni.showModal({
        title: '手动添加',
        content: '手动添加记录功能开发中...',
        showCancel: false
      })
    },

    // 查看记录详情
    viewRecordDetail(record) {
      uni.showModal({
        title: record.name,
        content: `地点：${record.location}\n日期：${record.date}\n评分：${record.rating}星\n\n详情功能开发中...`,
        showCancel: false
      })
    },

    // 查看分类记录
    viewCategoryRecords(category) {
      uni.showModal({
        title: category.name,
        content: `${category.name}分类有${category.count}条记录\n\n分类查看功能开发中...`,
        showCancel: false
      })
    },

    // 查看健康档案
    viewHealthProfile() {
      uni.showModal({
        title: '健康档案',
        content: '健康档案功能开发中，将包含个人健康状况、过敏信息、饮食偏好等。',
        showCancel: false
      })
    },

    // 查看设置
    viewSettings() {
      uni.showModal({
        title: '设置',
        content: '设置功能开发中，将包含通知设置、隐私设置、主题设置等。',
        showCancel: false
      })
    },

    // 查看帮助
    viewHelp() {
      uni.showModal({
        title: '帮助与反馈',
        content: '帮助与反馈功能开发中，将提供使用指南和意见反馈渠道。',
        showCancel: false
      })
    },

    // 关于我们
    viewAbout() {
      uni.showModal({
        title: '关于我们',
        content: '今天吃什么 v1.0.0\n\n一个帮助您健康饮食的智能助手应用。',
        showCancel: false
      })
    },

    // 退出登录
    logout() {
      uni.showModal({
        title: '退出登录',
        content: '确定要退出当前账号吗？',
        success: (res) => {
          if (res.confirm) {
            // 清除登录信息
            uni.removeStorageSync('token')
            uni.removeStorageSync('userInfo')
            
            // 跳转到登录页
            uni.reLaunch({
              url: '/pages/login/login'
            })
          }
        }
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

/* 个人信息卡片 */
.profile-card {
  background: white;
  margin: 20rpx;
  border-radius: 20rpx;
  padding: 40rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.08);
}

.profile-header {
  display: flex;
  margin-bottom: 40rpx;
}

.avatar-large {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  margin-right: 30rpx;
}

.user-details {
  flex: 1;
}

.username {
  font-size: 36rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 12rpx;
}

.user-desc {
  font-size: 26rpx;
  color: #999;
  display: block;
  margin-bottom: 20rpx;
}

.edit-btn {
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 20rpx;
  font-size: 24rpx;
}

.stats-row {
  display: flex;
  justify-content: space-around;
  border-top: 1rpx solid #F0F0F0;
  padding-top: 30rpx;
}

.stat-item {
  text-align: center;
}

.stat-number {
  font-size: 40rpx;
  font-weight: 600;
  color: #FF6B6B;
  display: block;
  margin-bottom: 8rpx;
}

.stat-label {
  font-size: 24rpx;
  color: #999;
}

/* 通用区块样式 */
.section {
  margin: 30rpx 20rpx;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
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

/* 记录列表 */
.records-list {
  background: white;
  border-radius: 16rpx;
  overflow: hidden;
}

.record-item {
  display: flex;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1rpx solid #F0F0F0;
  position: relative;
}

.record-item:last-child {
  border-bottom: none;
}

.record-image {
  width: 100rpx;
  height: 100rpx;
  border-radius: 12rpx;
  margin-right: 20rpx;
}

.record-info {
  flex: 1;
}

.record-name {
  font-size: 28rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.record-location {
  font-size: 24rpx;
  color: #999;
  display: block;
  margin-bottom: 12rpx;
}

.record-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.record-date {
  font-size: 22rpx;
  color: #999;
}

.record-rating {
  display: flex;
}

.star {
  font-size: 20rpx;
  color: #DDD;
  margin-right: 2rpx;
}

.star.active {
  color: #FFD700;
}

.record-category {
  position: absolute;
  top: 20rpx;
  right: 20rpx;
}

.category-tag {
  font-size: 20rpx;
  padding: 4rpx 12rpx;
  border-radius: 12rpx;
  background: #F0F0F0;
  color: #666;
}

.category-tag.special {
  background: #FFE8E8;
  color: #FF6B6B;
}

.category-tag.home {
  background: #E8F5E8;
  color: #4CAF50;
}

.category-tag.tea {
  background: #FFF8E1;
  color: #FF9800;
}

.category-tag.hotpot {
  background: #FFEBEE;
  color: #F44336;
}

.category-tag.expensive {
  background: #F3E5F5;
  color: #9C27B0;
}

.category-tag.affordable {
  background: #E3F2FD;
  color: #2196F3;
}

/* 分类网格 */
.category-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 20rpx;
}

.category-item {
  background: white;
  padding: 30rpx;
  border-radius: 16rpx;
  text-align: center;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.category-icon {
  font-size: 48rpx;
  margin-bottom: 16rpx;
}

.category-name {
  font-size: 26rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.category-count {
  font-size: 20rpx;
  color: #999;
}

/* 菜单列表 */
.menu-list {
  background: white;
  border-radius: 16rpx;
  overflow: hidden;
}

.menu-item {
  display: flex;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1rpx solid #F0F0F0;
}

.menu-item:last-child {
  border-bottom: none;
}

.menu-icon {
  font-size: 40rpx;
  margin-right: 20rpx;
}

.menu-text {
  flex: 1;
  font-size: 28rpx;
  color: #333;
}

.menu-arrow {
  font-size: 24rpx;
  color: #999;
}

/* 退出登录 */
.logout-section {
  padding: 40rpx 20rpx;
}

.logout-btn {
  width: 100%;
  height: 100rpx;
  background: #FF4444;
  color: white;
  border: none;
  border-radius: 16rpx;
  font-size: 32rpx;
  font-weight: 600;
}
</style> 