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
            <span class="station-code">编号: {{ station.code }}</span>
          </div>
          
          <div class="card-content">
            <div class="location-info">
              <p class="address">📍 {{ formatAddress(station.addr) }}</p>
              <div class="coordinates">
                <span>🌐 经度: {{ station.longitude }}°</span>
                <span>🌐 纬度: {{ station.latitude }}°</span>
                <span>⛰️ 海拔: {{ station.height }}m</span>
              </div>
            </div>
            
            <div v-if="station.mp4" class="video-section">
              <video 
                :src="getVideoUrl(station.mp4)" 
                controls 
                preload="metadata"
                class="weather-video"
                @error="(e) => console.warn('视频加载失败:', station.name, e)"
              >
                您的浏览器不支持视频播放
              </video>
            </div>
            
            <div v-if="station.obtid" class="additional-info">
              <span class="obtid">🆔 观测站ID: {{ station.obtid }}</span>
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
  background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.app-header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 1.5rem 2rem;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.app-header h1 {
  margin: 0;
  color: #2d3436;
  font-size: 2rem;
  font-weight: 700;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.refresh-btn {
  background: #00b894;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 184, 148, 0.3);
}

.refresh-btn:hover:not(:disabled) {
  background: #00a085;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 184, 148, 0.4);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.update-time {
  color: #636e72;
  font-size: 0.9rem;
  background: rgba(255, 255, 255, 0.8);
  padding: 0.5rem 1rem;
  border-radius: 15px;
}

.app-main {
  padding: 2rem;
  min-height: calc(100vh - 200px);
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
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error {
  text-align: center;
  color: white;
  background: rgba(231, 76, 60, 0.9);
  padding: 2rem;
  border-radius: 15px;
  margin: 2rem auto;
  max-width: 500px;
}

.retry-btn {
  background: white;
  color: #e74c3c;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  margin-top: 1rem;
  transition: all 0.3s ease;
}

.retry-btn:hover {
  background: #f8f9fa;
  transform: translateY(-2px);
}

.weather-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
  gap: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.weather-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 1.5rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.weather-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 2px solid #f1f2f6;
}

.card-header h3 {
  margin: 0;
  color: #2d3436;
  font-size: 1.4rem;
  font-weight: 700;
}

.station-code {
  background: #74b9ff;
  color: white;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
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

.coordinates {
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
  font-size: 0.85rem;
}

.coordinates span {
  background: #f8f9fa;
  color: #2d3436;
  padding: 0.4rem 0.8rem;
  border-radius: 12px;
  border: 1px solid #e9ecef;
}

.video-section {
  margin: 1.5rem 0;
}

.weather-video {
  width: 100%;
  max-height: 250px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.additional-info {
  margin-top: 1rem;
}

.obtid {
  background: #00b894;
  color: white;
  padding: 0.4rem 0.8rem;
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
}

.no-data {
  text-align: center;
  color: white;
  font-size: 1.2rem;
  margin-top: 3rem;
}

.app-footer {
  background: rgba(45, 52, 54, 0.9);
  color: white;
  text-align: center;
  padding: 1.5rem;
  margin-top: 2rem;
}

.app-footer p {
  margin: 0;
  font-size: 0.9rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .app-header {
    padding: 1rem;
    flex-direction: column;
    text-align: center;
  }
  
  .app-header h1 {
    font-size: 1.5rem;
  }
  
  .weather-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .app-main {
    padding: 1rem;
  }
  
  .coordinates {
    flex-direction: column;
  }
}

@media (max-width: 480px) {
  .weather-card {
    padding: 1rem;
  }
  
  .card-header {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }
}
</style>
