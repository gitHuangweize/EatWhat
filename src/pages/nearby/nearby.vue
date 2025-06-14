<template>
  <view class="container">
    <!-- 搜索栏 -->
    <view class="search-bar">
      <view class="search-input">
        <text class="search-icon">🔍</text>
        <input 
          class="input" 
          placeholder="搜索附近的美食..." 
          v-model="searchKeyword"
          @input="onSearchInput"
        />
        <button class="filter-btn" @click="showFilterModal">
          <text class="filter-icon">⚙️</text>
        </button>
      </view>
    </view>

    <!-- 分类标签 -->
    <view class="category-tabs">
      <scroll-view class="tabs-scroll" scroll-x="true" show-scrollbar="false">
        <view class="tabs-list">
          <view 
            class="tab-item" 
            v-for="category in foodCategories" 
            :key="category.key"
            :class="{ active: selectedCategory === category.key }"
            @click="selectCategory(category.key)"
          >
            <text class="tab-icon">{{ category.icon }}</text>
            <text class="tab-text">{{ category.name }}</text>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 推荐餐厅 -->
    <view class="recommendation-section">
      <view class="section-header">
        <text class="section-title">🎯 推荐餐厅</text>
        <text class="section-subtitle">根据您的口味和距离</text>
      </view>
      
      <scroll-view class="recommendation-scroll" scroll-x="true" show-scrollbar="false">
        <view class="recommendation-list">
          <view 
            class="recommendation-item" 
            v-for="restaurant in recommendedRestaurants" 
            :key="restaurant.id"
            @click="viewRestaurantDetail(restaurant)"
          >
            <image class="restaurant-image" :src="restaurant.image" mode="aspectFill"></image>
            <view class="restaurant-info">
              <text class="restaurant-name">{{ restaurant.name }}</text>
              <view class="restaurant-meta">
                <text class="rating">⭐ {{ restaurant.rating }}</text>
                <text class="distance">📍 {{ restaurant.distance }}</text>
              </view>
              <text class="specialties">{{ restaurant.specialties.join('、') }}</text>
            </view>
            <view class="restaurant-badge" v-if="restaurant.badge">
              <text class="badge-text">{{ restaurant.badge }}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 附近餐厅列表 -->
    <view class="restaurants-section">
      <view class="section-header">
        <text class="section-title">附近餐厅</text>
        <view class="sort-options">
          <text 
            class="sort-option" 
            v-for="sort in sortOptions" 
            :key="sort.key"
            :class="{ active: selectedSort === sort.key }"
            @click="selectSort(sort.key)"
          >
            {{ sort.name }}
          </text>
        </view>
      </view>
      
      <view class="restaurants-list">
        <view 
          class="restaurant-card" 
          v-for="restaurant in nearbyRestaurants" 
          :key="restaurant.id"
          @click="viewRestaurantDetail(restaurant)"
        >
          <image class="card-image" :src="restaurant.image" mode="aspectFill"></image>
          <view class="card-content">
            <view class="card-header">
              <text class="card-title">{{ restaurant.name }}</text>
              <view class="card-rating">
                <text class="rating-score">{{ restaurant.rating }}</text>
                <text class="rating-star">⭐</text>
              </view>
            </view>
            
            <text class="card-category">{{ restaurant.category }}</text>
            
            <view class="card-tags">
              <text 
                class="tag" 
                v-for="tag in restaurant.tags" 
                :key="tag"
              >
                {{ tag }}
              </text>
            </view>
            
            <view class="card-footer">
              <text class="price-range">{{ restaurant.priceRange }}</text>
              <text class="distance">{{ restaurant.distance }}</text>
              <text class="delivery-time" v-if="restaurant.deliveryTime">
                🚚 {{ restaurant.deliveryTime }}
              </text>
            </view>
          </view>
          
          <view class="card-actions">
            <button class="action-btn" @click.stop="callRestaurant(restaurant)">
              📞
            </button>
            <button class="action-btn" @click.stop="getDirection(restaurant)">
              🧭
            </button>
            <button class="action-btn" @click.stop="toggleFavorite(restaurant)">
              {{ restaurant.isFavorite ? '❤️' : '🤍' }}
            </button>
          </view>
        </view>
      </view>
    </view>

    <!-- 加载更多 -->
    <view class="load-more" v-if="hasMore">
      <button class="load-more-btn" @click="loadMore" :disabled="loading">
        {{ loading ? '加载中...' : '加载更多' }}
      </button>
    </view>

    <!-- 筛选弹窗 -->
    <uni-popup ref="filterPopup" type="bottom" :safe-area="false">
      <view class="filter-popup">
        <view class="popup-header">
          <text class="popup-title">筛选条件</text>
          <button class="close-btn" @click="closeFilterModal">×</button>
        </view>
        
        <view class="filter-content">
          <view class="filter-group">
            <text class="filter-label">价格区间</text>
            <view class="filter-options">
              <text 
                class="filter-option" 
                v-for="price in priceOptions" 
                :key="price.key"
                :class="{ active: selectedPrice === price.key }"
                @click="selectPrice(price.key)"
              >
                {{ price.name }}
              </text>
            </view>
          </view>
          
          <view class="filter-group">
            <text class="filter-label">距离</text>
            <view class="filter-options">
              <text 
                class="filter-option" 
                v-for="distance in distanceOptions" 
                :key="distance.key"
                :class="{ active: selectedDistance === distance.key }"
                @click="selectDistance(distance.key)"
              >
                {{ distance.name }}
              </text>
            </view>
          </view>
          
          <view class="filter-group">
            <text class="filter-label">评分</text>
            <view class="filter-options">
              <text 
                class="filter-option" 
                v-for="rating in ratingOptions" 
                :key="rating.key"
                :class="{ active: selectedRating === rating.key }"
                @click="selectRating(rating.key)"
              >
                {{ rating.name }}
              </text>
            </view>
          </view>
        </view>
        
        <view class="popup-footer">
          <button class="reset-btn" @click="resetFilters">重置</button>
          <button class="confirm-btn" @click="applyFilters">确定</button>
        </view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
export default {
  data() {
    return {
      searchKeyword: '',
      selectedCategory: 'all',
      selectedSort: 'distance',
      selectedPrice: 'all',
      selectedDistance: 'all',
      selectedRating: 'all',
      loading: false,
      hasMore: true,
      
      foodCategories: [
        { key: 'all', name: '全部', icon: '🍽️' },
        { key: 'chinese', name: '中餐', icon: '🥢' },
        { key: 'western', name: '西餐', icon: '🍽️' },
        { key: 'fastfood', name: '快餐', icon: '🍔' },
        { key: 'hotpot', name: '火锅', icon: '🔥' },
        { key: 'bbq', name: '烧烤', icon: '🍖' },
        { key: 'dessert', name: '甜品', icon: '🍰' },
        { key: 'coffee', name: '咖啡', icon: '☕' }
      ],
      
      sortOptions: [
        { key: 'distance', name: '距离' },
        { key: 'rating', name: '评分' },
        { key: 'price', name: '价格' },
        { key: 'popularity', name: '人气' }
      ],
      
      priceOptions: [
        { key: 'all', name: '不限' },
        { key: 'low', name: '￥ (人均30以下)' },
        { key: 'medium', name: '￥￥ (人均30-80)' },
        { key: 'high', name: '￥￥￥ (人均80以上)' }
      ],
      
      distanceOptions: [
        { key: 'all', name: '不限' },
        { key: '500m', name: '500米内' },
        { key: '1km', name: '1公里内' },
        { key: '3km', name: '3公里内' },
        { key: '5km', name: '5公里内' }
      ],
      
      ratingOptions: [
        { key: 'all', name: '不限' },
        { key: '4.5', name: '4.5分以上' },
        { key: '4.0', name: '4.0分以上' },
        { key: '3.5', name: '3.5分以上' }
      ],
      
      recommendedRestaurants: [
        {
          id: 1,
          name: '蜀香园川菜馆',
          image: '/static/restaurants/sichuan.jpg',
          rating: 4.8,
          distance: '500m',
          specialties: ['麻婆豆腐', '宫保鸡丁', '水煮鱼'],
          badge: '口味推荐'
        },
        {
          id: 2,
          name: '老北京涮羊肉',
          image: '/static/restaurants/hotpot.jpg',
          rating: 4.6,
          distance: '800m',
          specialties: ['涮羊肉', '手切羊肉', '老北京铜锅'],
          badge: '距离最近'
        }
      ],
      
      nearbyRestaurants: [
        {
          id: 1,
          name: '蜀香园川菜馆',
          image: '/static/restaurants/sichuan.jpg',
          category: '川菜',
          rating: 4.8,
          distance: '500m',
          priceRange: '￥￥ 人均65元',
          deliveryTime: '30分钟',
          tags: ['麻辣', '下饭', '聚餐'],
          isFavorite: false,
          phone: '13800138000'
        },
        {
          id: 2,
          name: '老北京涮羊肉',
          image: '/static/restaurants/hotpot.jpg',
          category: '火锅',
          rating: 4.6,
          distance: '800m',
          priceRange: '￥￥￥ 人均88元',
          deliveryTime: '不支持外卖',
          tags: ['清汤', '羊肉', '传统'],
          isFavorite: true,
          phone: '13800138001'
        },
        {
          id: 3,
          name: '星巴克咖啡',
          image: '/static/restaurants/starbucks.jpg',
          category: '咖啡',
          rating: 4.4,
          distance: '1.2km',
          priceRange: '￥￥ 人均45元',
          deliveryTime: '25分钟',
          tags: ['咖啡', '轻食', '环境好'],
          isFavorite: false,
          phone: '13800138002'
        }
      ]
    }
  },

  onLoad() {
    this.getCurrentLocation()
    this.loadNearbyRestaurants()
  },

  methods: {
    // 获取当前位置
    getCurrentLocation() {
      uni.getLocation({
        type: 'gcj02',
        success: (res) => {
          console.log('当前位置:', res)
          // 这里可以根据位置加载附近餐厅
        },
        fail: () => {
          uni.showToast({
            title: '获取位置失败',
            icon: 'none'
          })
        }
      })
    },

    // 搜索输入
    onSearchInput() {
      // 防抖处理
      clearTimeout(this.searchTimer)
      this.searchTimer = setTimeout(() => {
        this.searchRestaurants()
      }, 500)
    },

    // 搜索餐厅
    searchRestaurants() {
      if (!this.searchKeyword.trim()) {
        this.loadNearbyRestaurants()
        return
      }
      
      uni.showLoading({
        title: '搜索中...'
      })
      
      // 模拟搜索
      setTimeout(() => {
        uni.hideLoading()
        // 这里应该根据关键词搜索餐厅
        console.log('搜索关键词:', this.searchKeyword)
      }, 1000)
    },

    // 选择分类
    selectCategory(categoryKey) {
      this.selectedCategory = categoryKey
      this.loadNearbyRestaurants()
    },

    // 选择排序
    selectSort(sortKey) {
      this.selectedSort = sortKey
      this.loadNearbyRestaurants()
    },

    // 加载附近餐厅
    loadNearbyRestaurants() {
      this.loading = true
      
      // 模拟API调用
      setTimeout(() => {
        this.loading = false
        // 这里应该根据筛选条件加载餐厅数据
      }, 1000)
    },

    // 加载更多
    loadMore() {
      if (this.loading) return
      
      this.loading = true
      
      // 模拟加载更多数据
      setTimeout(() => {
        this.loading = false
        // 这里应该加载更多餐厅数据
        // 如果没有更多数据，设置 hasMore = false
      }, 1500)
    },

    // 查看餐厅详情
    viewRestaurantDetail(restaurant) {
      uni.showModal({
        title: restaurant.name,
        content: `评分：${restaurant.rating}⭐\n距离：${restaurant.distance}\n价格：${restaurant.priceRange}\n\n餐厅详情功能开发中...`,
        showCancel: false
      })
    },

    // 拨打电话
    callRestaurant(restaurant) {
      uni.makePhoneCall({
        phoneNumber: restaurant.phone,
        fail: () => {
          uni.showToast({
            title: '拨打失败',
            icon: 'none'
          })
        }
      })
    },

    // 获取导航
    getDirection(restaurant) {
      // 这里可以调用地图API获取导航
      uni.showToast({
        title: '导航功能开发中',
        icon: 'none'
      })
    },

    // 切换收藏状态
    toggleFavorite(restaurant) {
      restaurant.isFavorite = !restaurant.isFavorite
      
      uni.showToast({
        title: restaurant.isFavorite ? '已收藏' : '已取消收藏',
        icon: 'success'
      })
    },

    // 显示筛选弹窗
    showFilterModal() {
      this.$refs.filterPopup.open()
    },

    // 关闭筛选弹窗
    closeFilterModal() {
      this.$refs.filterPopup.close()
    },

    // 选择价格
    selectPrice(priceKey) {
      this.selectedPrice = priceKey
    },

    // 选择距离
    selectDistance(distanceKey) {
      this.selectedDistance = distanceKey
    },

    // 选择评分
    selectRating(ratingKey) {
      this.selectedRating = ratingKey
    },

    // 重置筛选
    resetFilters() {
      this.selectedPrice = 'all'
      this.selectedDistance = 'all'
      this.selectedRating = 'all'
    },

    // 应用筛选
    applyFilters() {
      this.closeFilterModal()
      this.loadNearbyRestaurants()
      
      uni.showToast({
        title: '筛选条件已应用',
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

/* 搜索栏 */
.search-bar {
  background: white;
  padding: 20rpx 30rpx;
  border-bottom: 1rpx solid #F0F0F0;
}

.search-input {
  display: flex;
  align-items: center;
  background: #F8F9FA;
  border-radius: 25rpx;
  padding: 0 30rpx;
  height: 80rpx;
}

.search-icon {
  font-size: 32rpx;
  color: #999;
  margin-right: 20rpx;
}

.input {
  flex: 1;
  font-size: 28rpx;
  border: none;
  background: transparent;
}

.filter-btn {
  background: transparent;
  border: none;
  padding: 0;
  margin-left: 20rpx;
}

.filter-icon {
  font-size: 32rpx;
  color: #FF6B6B;
}

/* 分类标签 */
.category-tabs {
  background: white;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #F0F0F0;
}

.tabs-scroll {
  white-space: nowrap;
}

.tabs-list {
  display: flex;
  padding: 0 30rpx;
}

.tab-item {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  margin-right: 40rpx;
  padding: 20rpx;
  border-radius: 12rpx;
  min-width: 120rpx;
}

.tab-item.active {
  background: #FFF0F0;
}

.tab-icon {
  font-size: 40rpx;
  margin-bottom: 8rpx;
}

.tab-text {
  font-size: 22rpx;
  color: #333;
}

.tab-item.active .tab-text {
  color: #FF6B6B;
}

/* 推荐区域 */
.recommendation-section {
  margin: 30rpx 20rpx;
}

.section-header {
  margin-bottom: 20rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 8rpx;
}

.section-subtitle {
  font-size: 24rpx;
  color: #999;
}

.recommendation-scroll {
  white-space: nowrap;
}

.recommendation-list {
  display: flex;
  padding-left: 10rpx;
}

.recommendation-item {
  display: inline-block;
  width: 280rpx;
  background: white;
  border-radius: 16rpx;
  margin-right: 20rpx;
  overflow: hidden;
  position: relative;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.restaurant-image {
  width: 100%;
  height: 160rpx;
}

.restaurant-info {
  padding: 20rpx;
}

.restaurant-name {
  font-size: 26rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 12rpx;
}

.restaurant-meta {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8rpx;
}

.rating,
.distance {
  font-size: 20rpx;
  color: #999;
}

.specialties {
  font-size: 20rpx;
  color: #666;
}

.restaurant-badge {
  position: absolute;
  top: 12rpx;
  right: 12rpx;
  background: #FF6B6B;
  color: white;
  font-size: 18rpx;
  padding: 4rpx 12rpx;
  border-radius: 12rpx;
}

/* 餐厅列表 */
.restaurants-section {
  margin: 30rpx 20rpx;
}

.restaurants-section .section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

.sort-options {
  display: flex;
}

.sort-option {
  font-size: 24rpx;
  color: #999;
  margin-left: 20rpx;
  padding: 8rpx 16rpx;
  border-radius: 16rpx;
}

.sort-option.active {
  background: #FF6B6B;
  color: white;
}

.restaurants-list {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.restaurant-card {
  background: white;
  border-radius: 16rpx;
  padding: 20rpx;
  display: flex;
  position: relative;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.card-image {
  width: 120rpx;
  height: 120rpx;
  border-radius: 12rpx;
  margin-right: 20rpx;
}

.card-content {
  flex: 1;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 8rpx;
}

.card-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #333;
}

.card-rating {
  display: flex;
  align-items: center;
}

.rating-score {
  font-size: 24rpx;
  color: #FF6B6B;
  margin-right: 4rpx;
}

.rating-star {
  font-size: 20rpx;
  color: #FFD700;
}

.card-category {
  font-size: 22rpx;
  color: #999;
  margin-bottom: 12rpx;
}

.card-tags {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 12rpx;
}

.tag {
  background: #F0F0F0;
  color: #666;
  font-size: 18rpx;
  padding: 4rpx 8rpx;
  border-radius: 8rpx;
  margin-right: 8rpx;
  margin-bottom: 4rpx;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price-range,
.distance,
.delivery-time {
  font-size: 20rpx;
  color: #999;
}

.card-actions {
  display: flex;
  flex-direction: column;
  gap: 10rpx;
  margin-left: 10rpx;
}

.action-btn {
  width: 60rpx;
  height: 60rpx;
  background: #F8F9FA;
  border: none;
  border-radius: 50%;
  font-size: 28rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 加载更多 */
.load-more {
  padding: 40rpx 20rpx;
  text-align: center;
}

.load-more-btn {
  width: 300rpx;
  height: 80rpx;
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 40rpx;
  font-size: 28rpx;
}

.load-more-btn:disabled {
  background: #ccc;
}

/* 筛选弹窗 */
.filter-popup {
  background: white;
  border-radius: 20rpx 20rpx 0 0;
  max-height: 80vh;
  overflow: hidden;
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1rpx solid #F0F0F0;
}

.popup-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
}

.close-btn {
  background: transparent;
  border: none;
  font-size: 48rpx;
  color: #999;
  padding: 0;
}

.filter-content {
  padding: 30rpx;
  max-height: 60vh;
  overflow-y: auto;
}

.filter-group {
  margin-bottom: 40rpx;
}

.filter-label {
  font-size: 28rpx;
  font-weight: 600;
  color: #333;
  display: block;
  margin-bottom: 20rpx;
}

.filter-options {
  display: flex;
  flex-wrap: wrap;
  gap: 20rpx;
}

.filter-option {
  background: #F8F9FA;
  color: #333;
  font-size: 24rpx;
  padding: 16rpx 24rpx;
  border-radius: 20rpx;
  border: 2rpx solid transparent;
}

.filter-option.active {
  background: #FFF0F0;
  color: #FF6B6B;
  border-color: #FF6B6B;
}

.popup-footer {
  display: flex;
  padding: 30rpx;
  gap: 20rpx;
  border-top: 1rpx solid #F0F0F0;
}

.reset-btn {
  flex: 1;
  height: 80rpx;
  background: #F8F9FA;
  color: #333;
  border: none;
  border-radius: 12rpx;
  font-size: 28rpx;
}

.confirm-btn {
  flex: 1;
  height: 80rpx;
  background: #FF6B6B;
  color: white;
  border: none;
  border-radius: 12rpx;
  font-size: 28rpx;
}
</style> 