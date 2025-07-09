<script setup>
import { ref, onMounted } from 'vue'

// 响应式数据
const weatherData = ref([])
const loading = ref(false)
const error = ref('')
const lastUpdateTime = ref('')

// 获取天气数据
const fetchWeatherData = async () => {
  loading.value = true
  error.value = ''

  try {
    const randomParam = Math.random()
    const timestamp = Date.now()
    const url = `/api/data_cache/video/rt/rt_list.js?random=${randomParam}&_=${timestamp}`

    const response = await fetch(url, {
      headers: {
        'accept': '*/*',
        'accept-language': 'zh-CN,zh;q=0.9',
        'cache-control': 'no-cache',
        'pragma': 'no-cache',
        'sec-ch-ua': '"Google Chrome";v="137", "Chromium";v="137", "Not/A)Brand";v="24"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"macOS"',
        'sec-fetch-dest': 'script',
        'sec-fetch-mode': 'no-cors',
        'sec-fetch-site': 'cross-site',
        'sec-fetch-storage-access': 'active'
      },
      referrer: 'https://weather.sz.gov.cn/',
      referrerPolicy: 'origin',
      method: 'GET',
      mode: 'cors',
      credentials: 'include'
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const text = await response.text()

    // 解析JSONP响应
    const match = text.match(/var SZ121_DATA = (\[.*?\]);/)
    if (match) {
      const data = JSON.parse(match[1])
      weatherData.value = data
      lastUpdateTime.value = new Date().toLocaleString('zh-CN')
    } else {
      throw new Error('无法解析数据格式')
    }
  } catch (err) {
    error.value = `获取数据失败: ${err.message}`
    console.error('获取天气数据失败:', err)
  } finally {
    loading.value = false
  }
}

// 组件挂载时获取数据
onMounted(() => {
  fetchWeatherData()
})

// 格式化地址显示
const formatAddress = (addr) => {
  return addr.trim().replace(/\s+/g, ' ')
}

// 获取视频完整URL
const getVideoUrl = (mp4Path) => {
  return `/api/data_cache/${mp4Path}`
}
</script>

<template>
  <div class="weather-app">
    <header class="app-header">
      <h1>🌤️ 深圳实时天气监控</h1>
      <div class="header-actions">
        <button @click="fetchWeatherData" :disabled="loading" class="refresh-btn">
          {{ loading ? '刷新中...' : '🔄 刷新数据' }}
        </button>
        <div v-if="lastUpdateTime" class="update-time">
          最后更新: {{ lastUpdateTime }}
        </div>
      </div>
    </header>

    <main class="app-main">
      <!-- 加载状态 -->
      <div v-if="loading" class="loading">
        <div class="loading-spinner"></div>
        <p>正在获取天气数据...</p>
      </div>

      <!-- 错误状态 -->
      <div v-else-if="error" class="error">
        <p>❌ {{ error }}</p>
        <button @click="fetchWeatherData" class="retry-btn">重试</button>
      </div>

      <!-- 天气数据展示 -->
      <div v-else-if="weatherData.length > 0" class="weather-grid">
        <div v-for="station in weatherData" :key="station.code" class="weather-card">
          <div class="card-header">
            <h3>{{ station.name }}</h3>
          </div>

          <div class="card-content">
            <div class="location-info">
              <p class="address">📍 {{ formatAddress(station.addr) }}</p>
              <div class="elevation">
                <span class="elevation-info">⛰️ 海拔: {{ station.height }}m</span>
              </div>
            </div>

            <div v-if="station.mp4" class="video-section">
              <video :src="getVideoUrl(station.mp4)" controls preload="metadata" class="weather-video"
                @error="(e) => console.warn('视频加载失败:', station.name, e)">
                您的浏览器不支持视频播放
              </video>
            </div>
          </div>
        </div>
      </div>

      <!-- 无数据状态 -->
      <div v-else class="no-data">
        <p>📭 暂无天气数据</p>
      </div>
    </main>

    <footer class="app-footer">
      <p>数据来源: 深圳市气象局 | 共 {{ weatherData.length }} 个监测站点</p>
    </footer>
  </div>
</template>

<style scoped>
.weather-app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  position: relative;
  overflow-x: hidden;
}

.weather-app::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background:
    radial-gradient(circle at 20% 80%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 255, 255, 0.1) 0%, transparent 50%),
    radial-gradient(circle at 40% 40%, rgba(120, 119, 198, 0.2) 0%, transparent 50%);
  pointer-events: none;
}

.app-header {
  background: rgba(255, 255, 255, 0.98);
  backdrop-filter: blur(20px);
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
  position: relative;
  z-index: 10;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.app-header h1 {
  margin: 0;
  color: #2d3436;
  font-size: 2.2rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.refresh-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 0.875rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.95rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
  position: relative;
  overflow: hidden;
}

.refresh-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.refresh-btn:hover::before {
  left: 100%;
}

.refresh-btn:hover:not(:disabled) {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 12px 35px rgba(102, 126, 234, 0.5);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.update-time {
  color: #636e72;
  font-size: 0.9rem;
  background: rgba(255, 255, 255, 0.9);
  padding: 0.75rem 1.25rem;
  border-radius: 25px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  font-weight: 500;
}

.app-main {
  width: 90%;
  max-width: 1600px;
  margin: 0 auto;
  padding: 3rem 2rem;
  min-height: calc(100vh - 200px);
  position: relative;
  z-index: 5;
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 300px;
  color: white;
  font-size: 1.2rem;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.error {
  text-align: center;
  color: white;
  background: rgba(231, 76, 60, 0.95);
  backdrop-filter: blur(20px);
  padding: 2.5rem;
  border-radius: 24px;
  margin: 2rem auto;
  max-width: 500px;
  box-shadow: 0 20px 60px rgba(231, 76, 60, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.retry-btn {
  background: white;
  color: #e74c3c;
  border: none;
  padding: 0.875rem 2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  margin-top: 1.5rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 8px 25px rgba(255, 255, 255, 0.3);
}

.retry-btn:hover {
  background: #f8f9fa;
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 12px 35px rgba(255, 255, 255, 0.4);
}

.weather-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(420px, 1fr));
  gap: 2.5rem;
  margin: 0 auto;
}

.weather-card {
  background: rgba(255, 255, 255, 0.98);
  backdrop-filter: blur(20px);
  border-radius: 24px;
  padding: 2rem;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.12);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.weather-card:hover {
  transform: translateY(-4px) scale(1.01);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1.5rem;
  border-bottom: 2px solid rgba(102, 126, 234, 0.1);
  position: relative;
}

.card-header h3 {
  margin: 0;
  color: #2d3436;
  font-size: 1.5rem;
  font-weight: 800;
  letter-spacing: -0.01em;
}



.card-content {
  space-y: 1rem;
}

.location-info {
  margin-bottom: 1.5rem;
}

.address {
  color: #636e72;
  margin: 0 0 0.8rem 0;
  font-size: 0.95rem;
  line-height: 1.4;
}

.elevation {
  margin-top: 0.5rem;
}

.elevation-info {
  background: rgba(255, 255, 255, 0.8);
  color: #2d3436;
  padding: 0.5rem 1rem;
  border-radius: 16px;
  border: 1px solid rgba(102, 126, 234, 0.1);
  backdrop-filter: blur(10px);
  font-weight: 500;
  font-size: 0.85rem;
  transition: all 0.3s ease;
  display: inline-block;
}

.elevation-info:hover {
  background: rgba(102, 126, 234, 0.1);
  transform: translateY(-1px);
}



.video-section {
  margin: 1.5rem 0 0 0;
}

.weather-video {
  width: 100%;
  max-height: 280px;
  border-radius: 16px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.15);
}



.no-data {
  text-align: center;
  color: white;
  font-size: 1.2rem;
  margin-top: 3rem;
}

.app-footer {
  background: rgba(45, 52, 54, 0.95);
  backdrop-filter: blur(20px);
  color: white;
  text-align: center;
  padding: 2rem;
  margin-top: 3rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  position: relative;
  z-index: 10;
}

.app-footer p {
  margin: 0;
  font-size: 0.9rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .app-header {
    padding: 1.5rem 1rem;
    flex-direction: column;
    text-align: center;
  }

  .app-header h1 {
    font-size: 1.8rem;
  }

  .weather-grid {
    grid-template-columns: 1fr;
    gap: 2rem;
  }

  .app-main {
    width: 95%;
    padding: 2rem 1rem;
  }



  .weather-card {
    padding: 1.5rem;
  }
}

@media (max-width: 480px) {
  .app-main {
    width: 98%;
    padding: 1.5rem 0.5rem;
  }

  .weather-card {
    padding: 1.25rem;
    border-radius: 20px;
  }

  .card-header {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
    margin-bottom: 1.25rem;
    padding-bottom: 1.25rem;
  }

  .app-header h1 {
    font-size: 1.6rem;
  }

  .weather-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
}
</style>
