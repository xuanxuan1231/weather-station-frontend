<template>
  <v-container class="fill-screen py-8">
    <v-row justify="center" align="center" class="mb-6">
      <v-col cols="12" md="8">
        <v-card class="scrollable-container mx-auto">
          <v-card-title class="text-h5">天气预览</v-card-title>
          <v-card-text>
            <v-alert v-if="error" type="error" class="mb-4">
              {{ errorMsg }}
            </v-alert>
            <v-row>
              <v-col cols="12" sm="6" md="3" v-for="item in categories" :key="item.key">
                <v-sheet class="pa-4 text-center" elevation="2">
                  <div class="text-h6 mb-2">{{ item.label }}</div>
                  <div v-if="item.key === 'temp'">
                    <span class="text-h4 font-weight-bold">{{ displayTemp }}</span>
                    <span class="ml-1">°{{ tempUnit }}</span>
                    <v-btn size="x-small" variant="text" @click="toggleTempUnit">切换单位</v-btn>
                  </div>
                  <div v-else>
                    <span class="text-h4 font-weight-bold">{{ weather[item.key] ?? '--' }}</span>
                    <span class="ml-1">{{ item.unit }}</span>
                  </div>
                  <div class="mt-2 text-caption grey--text" v-if="time[item.key]">
                    更新时间：{{ time[item.key] }}
                  </div>
                </v-sheet>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const categories = [
  { key: 'temp', label: '温度', unit: 'C' },
  { key: 'pressure', label: '大气压强', unit: 'hPa' },
  { key: 'pm2_5', label: 'PM2.5', unit: 'μg/m³' },
  { key: 'humidity', label: '湿度', unit: '%' },
]

const weather = ref({})
const tempUnit = ref('C')
const error = ref(false)
const errorMsg = ref('')
const time = ref({})

const fetchWeather = async (key) => {
  try {
    const res = await fetch(`https://wt-back.gemen.pp.ua/api/${key}`)
    let data
    try {
      data = await res.json()
    } catch (e) {
      weather.value[key] = '--'
      error.value = true
      errorMsg.value = '接口响应体解析失败'
      time.value[key] = '2000-1-1 00:00:00'
      return
    }
    if (data.status === 0) {
      weather.value[key] = data[key]
      if (data.time) {
        const utcDate = new Date(data.time)
        // 转为北京时间（UTC+8）
        const beijingDate = new Date(utcDate.getTime() + 8 * 60 * 60 * 1000)
        // 格式化为 YYYY-MM-DD HH:mm:ss
        const pad = n => n.toString().padStart(2, '0')
        time.value[key] = `${beijingDate.getFullYear()}-${pad(beijingDate.getMonth() + 1)}-${pad(beijingDate.getDate())} ${pad(beijingDate.getHours())}:${pad(beijingDate.getMinutes())}:${pad(beijingDate.getSeconds())}`
      }
    } else if (data.status === 1) {
      weather.value[key] = '--'
      error.value = true
      errorMsg.value = data.error || '接口返回错误'
      time.value[key] = '1970-1-1 08:00:00'
    } else {
      weather.value[key] = '--'
      error.value = true
      errorMsg.value = '未知错误'
      time.value[key] = '1970-1-1 08:00:00'
    }
  } catch (e) {
    weather.value[key] = '--'
    error.value = true
    errorMsg.value = e.message || '接口异常'
    time.value[key] = '1970-1-1 08:00:00'
  }
}

onMounted(() => {
  categories.forEach(cat => fetchWeather(cat.key))
})

const toggleTempUnit = () => {
  tempUnit.value = tempUnit.value === 'C' ? 'F' : 'C'
}

const displayTemp = computed(() => {
  const c = weather.value.temp
  if (c === undefined || c === '--') return '--'
  return tempUnit.value === 'C' ? c : (c * 9/5 + 32).toFixed(1)
})
</script>

<style scoped>
.fill-screen {
  min-height: 100vh;
  min-width: 100vw;
  height: 100vh;
  width: 100vw;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
}
.scrollable-container {
  overflow-y: auto;
  max-height: 100vh;
  width: 100%;
  box-sizing: border-box;
}

.text-h4 {
  font-size: 2rem;
}
</style>
