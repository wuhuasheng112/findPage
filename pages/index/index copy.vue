<template>
  <view class="discovery-page">
    <!-- 使用 uni-app 的下拉刷新和上拉加载 -->
    <scroll-view 
      class="scroll-container" 
      scroll-y 
      refresher-enabled 
      :refresher-triggered="isRefreshing"
      @refresherrefresh="onRefresh"
      @scrolltolower="onLoadMore"
    >
      <!-- 内容区域 -->
      <view class="content-container">
        <view class="waterfall-container">
          <!-- 左列 -->
          <view class="waterfall-column">
            <view 
              v-for="item in leftColumn" 
              :key="item.id" 
              class="content-card"
              @tap="onCardTap(item)"
            >
              <view class="card-media">
                <video 
                  v-if="item.type === 'video'" 
                  :id="'video-' + item.id"
                  :src="item.mediaUrl" 
                  class="media-content" 
                  :poster="item.coverUrl"
                  :show-center-play-btn="true"
                  :enable-progress-gesture="true"
                  :show-fullscreen-btn="true"
                  :show-play-btn="true"
                  :object-fit="'contain'"
                  
                  :controls="false"
                  @play="onVideoPlay(item.id)"
                ></video>
                <image 
                  v-else 
                  :src="item.mediaUrl" 
                  class="media-content" 
                  mode="widthFix"
                ></image>
                <view v-if="item.type === 'video'" class="video-indicator">
                  <text class="iconfont icon-play"></text>
                </view>
              </view>
              <view class="card-content">
                <text class="card-title">{{ item.title }}</text>
                <text class="card-tag">{{ item.tags[0] }}</text>
                <view class="card-info">
                  <view class="user-info">
                    <image :src="item.user.avatar" class="user-avatar"></image>
                    <text class="user-name">{{ item.user.name }}</text>
                  </view>
                  <view class="interaction"> 
                    <image src="/static/star.svg"></image>
                    <text class="like-count">{{ item.stats.likes }}</text>
                  </view>
                </view>
              </view>
            </view>
          </view>
          
          <!-- 右列 -->
          <view class="waterfall-column">
            <view 
              v-for="item in rightColumn" 
              :key="item.id" 
              class="content-card"
              @tap="onCardTap(item)"
            >
              <view class="card-media">
                <video 
                  v-if="item.type === 'video'" 
                  :id="'video-' + item.id"
                  :src="item.mediaUrl" 
                  class="media-content" 
                  :poster="item.coverUrl"
                  :controls="false"
                  :show-center-play-btn="true"
                  :enable-progress-gesture="true"
                  :show-fullscreen-btn="true"
                  :show-play-btn="true"
                  :object-fit="'contain'"
                  @play="onVideoPlay(item.id)"
                ></video>
                <image 
                  v-else 
                  :src="item.mediaUrl" 
                  class="media-content" 
                  mode="widthFix"
                ></image>
                <view v-if="item.type === 'video'" class="video-indicator">
                  <text class="iconfont icon-play"></text>
                </view>
              </view>
              <view class="card-content">
                <text class="card-title">{{ item.title }}</text>
                <text class="card-tag">{{ item.tags[0] }}</text>
                <view class="card-info">
                  <view class="user-info">
                    <image :src="item.user.avatar" class="user-avatar"></image>
                    <text class="user-name">{{ item.user.name }}</text>
                  </view>
                  <view class="interaction">
                    <image src="/static/star.svg"></image>
                    <text class="like-count">{{ item.stats.likes }}</text>
                  </view>
                </view>
              </view>
            </view>
          </view>
        </view>
      </view>
      
      <!-- 加载更多提示 -->
      <view class="load-more" v-if="!isEnd">
        <text v-if="isLoading">加载中...</text>
        <text v-else>上拉加载更多</text>
      </view>
      
      <!-- 到底提示 -->
      <view class="end-tip" v-if="isEnd">
        <text>- 已经到底啦 -</text>
      </view>
    </scroll-view>
  </view>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      contentList: [],
      page: 1,
      pageSize: 10,
      isLoading: false,
      isRefreshing: false,
      isEnd: false,
      dataFormat: 'json',
      currentPlayingVideoId: null, // 当前正在播放的视频ID
      videoContexts: {} // 存储视频上下文对象
    }
  },
  
  computed: {
    // 计算左右两列数据
    leftColumn() {
      return this.contentList.filter((_, index) => index % 2 === 0)
    },
    
    rightColumn() {
      return this.contentList.filter((_, index) => index % 2 === 1)
    },
    
    // 获取所有视频项
    videoItems() {
      return this.contentList.filter(item => item.type === 'video')
    }
  },
  
  onLoad(options) {
    // 检查URL参数
    if (options.format) {
      this.dataFormat = options.format
    }
    
    // 加载初始数据
    this.fetchData()
  },
  
  methods: {
    // 视频播放事件处理
    onVideoPlay(videoId) {
      // 如果当前已有正在播放的视频，且不是当前点击的视频，则暂停它
      if (this.currentPlayingVideoId && this.currentPlayingVideoId !== videoId) {
        this.pauseVideo(this.currentPlayingVideoId)
      }
      
      // 更新当前播放的视频ID
      this.currentPlayingVideoId = videoId
      
      // 创建并存储视频上下文
      if (!this.videoContexts[videoId]) {
        this.videoContexts[videoId] = uni.createVideoContext('video-' + videoId, this)
      }
    },
    
    // 暂停指定视频
    pauseVideo(videoId) {
      // 获取或创建视频上下文
      if (!this.videoContexts[videoId]) {
        this.videoContexts[videoId] = uni.createVideoContext('video-' + videoId, this)
      }
      
      // 暂停视频
      this.videoContexts[videoId].pause()
    },
    
    // 暂停所有视频，除了指定ID的视频
    pauseAllVideosExcept(exceptVideoId) {
      this.videoItems.forEach(item => {
        if (item.id !== exceptVideoId) {
          this.pauseVideo(item.id)
        }
      })
    },
    
    // 下拉刷新
    onRefresh() {
      // 暂停所有视频
      this.pauseAllVideosExcept(null)
      this.currentPlayingVideoId = null
      
      this.isRefreshing = true
      this.refreshData()
    },
    
    // 上拉加载更多
    onLoadMore() {
      if (!this.isLoading && !this.isEnd) {
        this.loadMoreData()
      }
    },
    
    // 点击卡片
    onCardTap(item) {
      // 这里可以实现跳转到详情页的逻辑
      uni.showToast({
        title: `点击了: ${item.title}`,
        icon: 'none'
      })
    },
    
    // 获取数据
    async fetchData(isRefresh = false) {
      if (isRefresh) {
        this.page = 1
        this.isEnd = false
      }
      
      this.isLoading = true
      
      try {
        
        const mockData = await axios.get('/static/mock.json?3')
		    const data = mockData.data.data 
        console.log('data',data);
        
        if (isRefresh) {
          this.contentList = data.items
          // 重置视频上下文
          this.videoContexts = {}
          this.currentPlayingVideoId = null
        } else {
          this.contentList = [...this.contentList, ...data.items]
        }
        
        // 判断是否还有更多数据
        this.isEnd = !data.pageInfo.hasMore
        
        this.page++
        
        // 在数据加载完成后，初始化视频上下文
        this.$nextTick(() => {
          this.initVideoContexts()
        })
      } catch (error) {
        console.error('获取数据失败:', error)
        uni.showToast({
          title: '获取数据失败',
          icon: 'none'
        })
      } finally {
        this.isLoading = false
        
        if (isRefresh) {
          this.isRefreshing = false
        }
      }
    },
    
    // 初始化视频上下文
    initVideoContexts() {
      this.videoItems.forEach(item => {
        if (!this.videoContexts[item.id]) {
          this.videoContexts[item.id] = uni.createVideoContext('video-' + item.id, this)
        }
      })
    },
    
    // 刷新数据
    refreshData() {
      this.fetchData(true)
    },
    
    // 加载更多数据
    loadMoreData() {
      if (!this.isEnd) {
        this.fetchData()
      }
    }
  },
  
  // 页面滚动时检测视频是否在可视区域
  onPageScroll(e) {
    // 这里可以添加视频自动播放/暂停逻辑
    // 需要计算视频元素是否在可视区域内
    // 由于UniApp不直接支持IntersectionObserver API，这部分逻辑较复杂
    // 可以考虑使用节流函数优化性能
  }
}
</script>

<style>
/* 样式保持不变 */
.discovery-page {
  width: 100%;
  height: 100vh;
  background-color: #f5f5f5;
}

.scroll-container {
  height: 100%;
}

.content-container {
  padding: 20rpx;
}

.waterfall-container {
  display: flex;
  justify-content: space-between;
}

.waterfall-column {
  width: 49%;
}

.content-card {
  margin-bottom: 20rpx;
  background-color: #fff;
  border-radius: 16rpx;
  overflow: hidden;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
}

.card-media {
  position: relative;
  width: 100%;
}

.media-content {
  width: 100%;
  display: block;
}

.video-indicator {
  position: absolute;
  right: 20rpx;
  bottom: 20rpx;
  width: 60rpx;
  height: 60rpx;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.card-content {
  padding: 20rpx;
}

.card-title {
  margin: 0 0 10rpx;
  font-size: 28rpx;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
.card-tag{
  font-size: 24rpx;
  
  padding: 4rpx 8rpx;
  border-radius: 8rpx;
  color: #666;
  background-color: #f6f6f6;
  margin-bottom: 10rpx;
}
.card-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 24rpx;
  color: #666;
}

.user-info {
  display: flex;
  align-items: center;
}

.user-avatar {
  width: 40rpx;
  height: 40rpx;
  border-radius: 50%;
  margin-right: 10rpx;
}

.interaction {
  display: flex;
  align-items: center;
}
.interaction image{
  width: 32rpx;
  height: 32rpx;
}
.like-count {
  margin-left: 6rpx;
}

.load-more, .end-tip {
  height: 100rpx;
  line-height: 100rpx;
  text-align: center;
  color: #999;
  font-size: 28rpx;
}

/* 字体图标样式 */
@font-face {
  font-family: "iconfont";
  src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAOMAAsAAAAAB8AAAANAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDHAqCYIJGATYCJAMQCwoABCAFhG0HQRvSBhHVm1PIfiTYuCXEK6ZQhaLA4uHxNe/9vbvLzP4/EYQkGo1HJ5FIJCExCSKSSIVOqISGRvJ29P9um/8BjiVQABU4K7hgXdAECrY4rDgr8P+5nN4YfH7Lchlj0BjbJo4DjAMKaK+vBRRIAXmDsQte4GkCDUO8kFvF1T1gK2OcFYh7jg7YJspIGXJDvVBdsViJF9Tr06N0AZ6934+/IGxLUinGmTsPSyqQ/jP4eYDX/79uOgrA7c8MbhMVK4BMuFHqOycKwivRMPQbZXsX6OuV5Gfwc+/ngc8D/v9HtMrIqRrhH15ElRBVxuQusAupwc9AqYSfhxICP5cSEr+AA0XDzzMmbJAGDW8Ap7BzJZEQJbRWo9Fo7OzSbG3VbG7VbW3VRQiRl5eXnZ2dlZWVmZmZnp6ekpKSGP+/dSLhGsY/QpgZ/vjx48PDw/39/d3d3e3t7c3Nzb6+vu7u7s7OzgcGBrq6ujo7Ojs6Ojo7Ojs7O7u6urq7u3t6enp7e/v6+vr7+wcGBgYHB4eGhoaHh0dGRkZHR8fGxsbHxycmJianJqemZ6ZnZmfn5uYX5hcWl5aWV5ZXVtfW1jc2Nre2tnZ2dvb29g8ODo+Oj0/OL84vzs8v7u7uHx4enp6eX15e3t7ePjw8PD4+Pj09vby8vL6+vr29vX94eHh8fHx6enp+fn55eXl9fX17e3t/f//w8PD4+Pj09PT8/Pzy8vL6+vr29vb+/v7h4eHx8fHp6en5+fnl5eX19fXt7e39/f3D4+PT89PL68vr2+v7x8en56eX15fXt9f3j4+/AH5Z+WXtl/VfNn7Z/GXrl+1fdn7Z/WXvl/1fDn45/OX4l5NfTn85++X8l4tfLn+5+uX6l5tfbn+5++X+l4dfHn95+uX5l5dfXn95++X9l49fPn/5+uX7l59ffn/5++X/HwAGhQWFB0UERQZFBcUExQbFBSUEJQYlBaUEpQalBWUEZQZlBeUE5QblBRUEFQYVBZUElQaVBVUEVQZVBdUE1QbVBTUENQY1BTUHtQS1BrUFtQd1BHUGdQV1B/UE9Qb1BfUHDQQNBg0FDQeNBI0GjQWNB00ETQZNBc0EzQbNBc0HLQQtBi0FLQetBK0GrQWtB20EbQZtBW0H7QTtBu0F7QcdBB0GHQUdB50EnQadBZ0HXQRdBl0FXQfdBN0G3QXdBz0EPQYtAACGhgA=') format('woff2');
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 32rpx;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-play:before {
  content: "\e600";
}

.icon-star:before {
  content: "\e601";
}
</style>