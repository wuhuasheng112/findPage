<template>
    <view class="user-recommendations">
      <view class="recommendation-header">
        <text class="header-title">优质用户推荐</text>
        <view class="indicator-bar">
          <view class="indicator-active"></view>
          <view class="indicator-dot"></view>
          <view class="indicator-dot"></view>
        </view>
      </view>
      
      <view class="user-list">
        <view 
          v-for="(user, index) in recommendedUsers" 
          :key="user.id"
          class="user-item"
          @tap="onUserTap(user)"
        >
          <view class="user-avatar-container">
            <image :src="user.avatar" class="user-pic" mode="aspectFill"></image>
            <view  class="user-badge">
                +
            </view>
          </view>
          
          <view class="user-des">
            <view class="user-name">{{ user.name }}</view>
            <view class="user-description">{{ user.description }}</view>
          </view>
        </view>
      </view>
    </view>
  </template>
  
  <script>
  
  export default {
    name: 'UserRecommendations',
    props: {
      users: {
        type: Array,
        default: () => []
      }
    },
    data() {
      return {
        // 默认推荐用户数据，如果没有传入props则使用这些
        defaultUsers: [
          {
            id: 'user001',
            name: '阿萌',
            description: '不能无聊，凡事自己来的',
            avatar: 'https://randomuser.me/api/portraits/men/32.jpg',
            badge: ''
          },
          {
            id: 'user002',
            name: '青空千绪',
            description: '关注我，做我的人请举手！',
            avatar: 'https://randomuser.me/api/portraits/women/44.jpg',
            badge: '+'
          },
          {
            id: 'user003',
            name: '大橙无情',
            description: '喜欢橙子和柠檬，每天愉快',
            avatar: 'https://randomuser.me/api/portraits/men/85.jpg',
            badge: ''
          }
        ]
      }
    },
    computed: {
      recommendedUsers() {
        return this.users.length > 0 ? this.users : this.defaultUsers
      }
    },
    methods: {
      onUserTap(user) {
        this.$emit('user-tap', user)
        uni.showToast({
          title: `关注了: ${user.name}`,
          icon: 'none'
        })
      }
    }
  }
  </script>
  
  <style scoped>
  .user-recommendations {
    background-color: #fff;
    border-radius: 12rpx;
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
    padding: 20rpx;
    box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.05);
  }
  
  .recommendation-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 16rpx;
    border-bottom: 1rpx solid #f5f5f5;
  }
  
  .header-title {
    font-size: 28rpx;
    font-weight: bold;
    color: #333;
  }
  
  .indicator-bar {
    display: flex;
    align-items: center;
  }
  
  .indicator-active {
    width: 30rpx;
    height: 6rpx;
    background-color: #ff9500;
    border-radius: 3rpx;
    margin-right: 6rpx;
  }
  
  .indicator-dot {
    width: 6rpx;
    height: 6rpx;
    background-color: #ddd;
    border-radius: 50%;
    margin-right: 6rpx;
  }
  
  .user-list {
    padding-top: 16rpx;
  }
  
  .user-item {
    display: flex;
    align-items: center;
    padding: 16rpx 0;
  }
  
  .user-avatar-container {
    position: relative;
    margin-right: 16rpx;
  }
  
  .user-pic {
    width: 80rpx!important;
    height: 80rpx!important;
    border-radius: 50%;
    background-color: #f5f5f5;
  }
  
  .user-badge {
    position: absolute;
    /* 居中 */
    right: calc((100% - 32rpx) / 2);
    bottom: -4rpx;
    width: 32rpx;
    height: 24rpx;
    background-color: #ff9500;
    border-radius: 18rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    line-height: 18rpx;
    
    color: #fff;
    font-size: 24rpx;
    font-weight: bold;
  }
  
  .user-badge text {
  }
  
  .user-des {
    flex: 1;
  }
  
  .user-name {
    font-size: 28rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 6rpx;
  }
  
  .user-description {
    font-size: 24rpx;
    color: #999;
    line-height: 1.4;
  }
  </style>