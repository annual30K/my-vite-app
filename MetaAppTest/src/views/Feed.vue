<template>
  <div class="feed">
    <!-- 下拉刷新提示 -->
    <div class="refresh-tip" :style="{ height: pullHeight + 'px' }">
      {{ refreshText }}
    </div>

    <!-- 瀑布流容器 -->
    <div class="waterfall">
      <!-- 左列 -->
      <div class="column">
        <Card v-for="item in leftItems" :key="item.id" :item="item" />
      </div>
      <!-- 右列 -->
      <div class="column">
        <Card v-for="item in rightItems" :key="item.id" :item="item" />
      </div>
    </div>

    <!-- 加载提示 -->
    <div class="loading-tip" v-if="isLoading">
      正在加载更多...
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Card from '../components/Card.vue'

// 数据和状态
const items = ref([]) // 所有内容
const leftItems = ref([]) // 左列内容
const rightItems = ref([]) // 右列内容
const page = ref(1) // 当前页码
const isLoading = ref(false) // 加载状态

// 下拉刷新相关状态
const pullHeight = ref(0) // 下拉高度
const refreshText = ref('') // 刷新提示文字
const startY = ref(0) // 触摸起始Y坐标
const isPulling = ref(false) // 是否处于下拉状态

// 生成随机文字内容（40-80字）
const generateRandomContent = () => {
  const words = [
    '今天', '天气', '很好', '阳光', '明媚', '心情', '愉快', '出去', '散步', '感觉', '不错',
    '朋友', '一起', '吃饭', '聊天', '分享', '快乐', '生活', '美好', '工作', '顺利', '学习',
    '进步', '梦想', '努力', '实现', '未来', '可期', '喜欢', '音乐', '电影', '旅行', '美食',
    '放松', '享受', '时光', '珍惜', '每一天', '微笑', '面对', '挑战', '勇敢', '前行'
  ]
  const minWords = 10
  const maxWords = 15
  const wordCount = Math.floor(Math.random() * (maxWords - minWords + 1)) + minWords
  let content = ''
  for (let i = 0; i < wordCount; i++) {
    const randomWord = words[Math.floor(Math.random() * words.length)]
    content += randomWord
    if (i < wordCount - 1) content += ''
  }
  return content
}

// 模拟数据获取
const fetchData = async (isRefresh = false) => {
  if (isLoading.value) return

  isLoading.value = true
  try {
    const res = await new Promise(resolve => {
      setTimeout(() => {
        const newItems = Array.from({ length: 10 }, (_, i) => {
          const id = Date.now() + i
          const isVideo = Math.random() > 0.7 // 30% 概率为视频
          return {
            id,
            type: isVideo ? 'video' : 'image',
            src: isVideo
              ? 'https://www.w3schools.com/html/mov_bbb.mp4'
              : `https://picsum.photos/300/${Math.floor(Math.random() * 200 + 100)}?random=${id}`,
            thumbnail: isVideo
              ? `https://picsum.photos/300/200?random=${id}`
              : undefined,
            content: generateRandomContent() // 使用随机生成的文字内容
          }
        })
        resolve(newItems)
      }, 1000)
    })

    if (isRefresh) {
      items.value = res
      page.value = 1
    } else {
      items.value = [...items.value, ...res]
      page.value++
    }
    distributeItems()
  } finally {
    isLoading.value = false
  }
}

// 分配数据到两列
const distributeItems = () => {
  leftItems.value = []
  rightItems.value = []
  items.value.forEach((item, index) => {
    if (index % 2 === 0) {
      leftItems.value.push(item)
    } else {
      rightItems.value.push(item)
    }
  })
}

// 触摸事件处理（下拉刷新）
const handleTouchStart = (e) => {
  startY.value = e.touches[0].pageY
  isPulling.value = window.scrollY === 0
}

const handleTouchMove = (e) => {
  if (!isPulling.value) return

  const currentY = e.touches[0].pageY
  const diff = currentY - startY.value

  if (diff > 0) {
    e.preventDefault()
    pullHeight.value = Math.min(diff * 0.6, 80) // 阻尼效果，最大高度80px
    refreshText.value = pullHeight.value >= 80 ? '释放刷新' : '下拉刷新'
  }
}

const handleTouchEnd = async () => {
  if (pullHeight.value >= 80) {
    refreshText.value = '正在刷新...'
    await fetchData(true) // 触发刷新
    pullHeight.value = 0
    refreshText.value = ''
  } else {
    pullHeight.value = 0
  }
  isPulling.value = false
}

// 无限滚动
const handleScroll = () => {
  const scrollTop = document.documentElement.scrollTop || document.body.scrollTop
  const scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight
  const clientHeight = document.documentElement.clientHeight || document.body.clientHeight

  if (scrollHeight - scrollTop - clientHeight < 5 && !isLoading.value) {
    fetchData()
  }
}

// 初始化
onMounted(() => {
  fetchData() // 初次加载
  window.addEventListener('scroll', handleScroll)
  // 添加触摸事件监听
  document.addEventListener('touchstart', handleTouchStart, { passive: false })
  document.addEventListener('touchmove', handleTouchMove, { passive: false })
  document.addEventListener('touchend', handleTouchEnd)
})

// 清理
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  // 移除触摸事件监听
  document.removeEventListener('touchstart', handleTouchStart)
  document.removeEventListener('touchmove', handleTouchMove)
  document.removeEventListener('touchend', handleTouchEnd)
})
</script>

<style scoped>
.feed {
  padding: 10px;
  overflow-x: hidden;
  -webkit-overflow-scrolling: touch;
}

.refresh-tip {
  text-align: center;
  color: #666;
  font-size: 14px;
  transition: height 0.3s;
}

.waterfall {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  padding: 5px;
}

.column {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.loading-tip {
  text-align: center;
  padding: 15px;
  color: #666;
  font-size: 14px;
}

/* 移动端保持双列 */
@media (max-width: 768px) {
  .feed {
    padding: 5px; /* 移动端减少外边距 */
  }
  .waterfall {
    gap: 10px; /* 移动端减少列间距 */
  }
  .column {
    flex: 1; /* 保持双列 */
  }
}
</style>