<template>
  <div class="card">
    <!-- 媒体区域 -->
    <div class="media-wrapper">
      <!-- 视频 -->
      <div v-if="item.type === 'video'" class="video-container">
        <video
          v-if="isMediaLoaded"
          ref="videoPlayer"
          :src="item.src"
          :poster="item.thumbnail"
          class="media"
          preload="metadata"
          playsinline
          webkit-playsinline
          x5-video-player-type="h5"
          @loadstart="onMediaLoad"
          @error="onMediaError"
        />
        <!-- 骨架屏占位（视频） -->
        <div v-else class="skeleton-media"></div>
        <!-- 播放图标 -->
        <div v-if="isMediaLoaded" class="play-icon" @click="playVideo">
          <svg viewBox="0 0 24 24" class="play-icon-svg">
            <path d="M8 5v14l11-7z" fill="white" />
          </svg>
        </div>
      </div>
      <!-- 图片 -->
      <img
        v-else-if="item.type === 'image' && isMediaLoaded"
        :src="item.src"
        alt=""
        class="media"
        loading="lazy"
        @load="onMediaLoad"
        @error="onMediaError"
      />
      <!-- 骨架屏占位（图片） -->
      <div v-else-if="item.type === 'image'" class="skeleton-media"></div>
    </div>
    <div class="content">
      <!-- 文字内容也使用骨架屏占位 -->
      <div v-if="!isTextLoaded" class="skeleton-text"></div>
      <p v-else>{{ item.content }}</p>
    </div>

    <!-- 页面内全屏播放遮罩 -->
    <div v-if="isPlaying" class="video-overlay" @click="stopVideo">
      <video
        ref="overlayVideo"
        :src="item.src"
        class="overlay-video"
        preload="metadata"
        playsinline
        webkit-playsinline
        x5-video-player-type="h5"
        @error="onOverlayVideoError"
        @loadedmetadata="onOverlayVideoLoaded"
      />
      <div class="close-icon">
        <svg viewBox="0 0 24 24" class="close-icon-svg">
          <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" fill="white" />
        </svg>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'

const props = defineProps({
  item: {
    type: Object,
    required: true
  }
})

const isMediaLoaded = ref(false) // 标记媒体（图片/视频）是否加载完成
const isTextLoaded = ref(true) // 标记文字是否加载完成（假设文字立即可用）
const isPlaying = ref(false) // 标记是否正在播放（页面内全屏）
const videoPlayer = ref(null) // 视频元素引用
const overlayVideo = ref(null) // 全屏播放视频元素引用

// 媒体加载完成
const onMediaLoad = () => {
  console.log('Media loaded:', props.item.src)
  isMediaLoaded.value = true
}

// 媒体加载失败
const onMediaError = () => {
  console.warn('Media failed to load:', props.item.src)
  isMediaLoaded.value = true // 加载失败时也显示内容，避免卡在骨架屏
}

// 遮罩层视频加载完成
const onOverlayVideoLoaded = () => {
  console.log('Overlay video loaded metadata:', props.item.src)
  if (overlayVideo.value) {
    overlayVideo.value.play().catch(error => {
      console.error('Overlay video play failed:', error)
    })
  }
}

// 遮罩层视频加载失败
const onOverlayVideoError = () => {
  console.error('Overlay video failed to load:', props.item.src)
}

// 超时机制，防止加载时间过长
onMounted(() => {
  // 检查 item.src 是否存在
  if (!props.item.src) {
    console.warn('Media src is not provided:', props.item)
    isMediaLoaded.value = true // 如果 src 为空，直接显示（可能显示破图）
    return
  }

  setTimeout(() => {
    if (!isMediaLoaded.value) {
      console.warn('Media load timeout:', props.item.src)
      isMediaLoaded.value = true // 3秒后强制显示内容
    }
  }, 3000)
})

// 播放视频（页面内全屏）
const playVideo = async () => {
  if (videoPlayer.value) {
    isPlaying.value = true // 显示遮罩层
    await nextTick() // 等待 DOM 更新
    if (overlayVideo.value) {
      overlayVideo.value.play().catch(error => {
        console.error('Overlay video play failed:', error)
      })
    }
  }
}

// 停止播放并关闭遮罩
const stopVideo = () => {
  if (overlayVideo.value) {
    overlayVideo.value.pause() // 暂停视频
    overlayVideo.value.currentTime = 0 // 重置播放进度
  }
  isPlaying.value = false // 隐藏遮罩层
}
</script>

<style scoped>
.card {
  background: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.media-wrapper {
  position: relative;
  padding-top: 56.25%; /* 16:9 宽高比，占位高度 */
}

.video-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.media {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover; /* 确保视频或图片填满容器 */
}

.skeleton-media {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #f0f0f0;
  animation: skeleton-loading 1.5s infinite;
}

.play-icon {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 40px;
  height: 40px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.3s;
}

.play-icon:hover {
  background: rgba(0, 0, 0, 0.7);
}

.play-icon-svg {
  width: 20px;
  height: 20px;
}

.content {
  padding: 10px;
  font-size: 14px;
  color: #333;
}

.skeleton-text {
  height: 14px;
  background: #f0f0f0;
  border-radius: 4px;
  animation: skeleton-loading 1.5s infinite;
  margin: 5px 0;
}

/* 页面内全屏播放遮罩 */
.video-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.overlay-video {
  width: 100%;
  height: 100%;
  object-fit: contain; /* 视频在全屏时保持比例，不裁剪 */
}

.close-icon {
  position: absolute;
  top: 20px;
  right: 20px;
  width: 40px;
  height: 40px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.3s;
}

.close-icon:hover {
  background: rgba(0, 0, 0, 0.7);
}

.close-icon-svg {
  width: 24px;
  height: 24px;
}

/* 骨架屏闪烁动画 */
@keyframes skeleton-loading {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}
</style>