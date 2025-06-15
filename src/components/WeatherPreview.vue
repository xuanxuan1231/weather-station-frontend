<template>
  <v-container class="fill-screen py-8">
    <v-row justify="center" align="center" class="mb-6">
      <v-col cols="12" md="8">
        <v-card class="scrollable-container mx-auto card">
          <v-card-title class="text-h5">
            天气预报<br>
            <span class="text-caption ml-4">{{ countdown }} 秒后刷新</span>
          </v-card-title>
          <v-card-text>
            <v-alert v-if="error" type="error" class="mb-4">
              {{ errorMsg }}
            </v-alert>
            <v-row>
              <v-col cols="12" sm="6" md="3" v-for="item in categories" :key="item.key">
                <v-sheet class="pa-4 text-center card" elevation="2">
                  <div class="weather-icon-wrapper">
                    <component :is="item.icon" class="weather-icon" />
                  </div>

                  <div v-if="item.key === 'temp'">
                    <div class="text-h6 mb-2 temp-label">{{ item.label }}</div>
                    <div class="value-unit temp">
                      <span class="temp-value">{{ displayTemp }}</span>
                      <span class="temp-unit">°{{ tempUnit }}</span>
                    </div>
                    <div class="unit-switcher mt-2">
                      <v-btn size="x-small" variant="text" @click="toggleTempUnit">切换单位</v-btn>
                    </div>
                    <div class="mt-2 text-caption grey--text no-margin-top" v-if="time[item.key]">
                      更新时间：{{ time[item.key] }}
                    </div>
                  </div>
                  <div v-else>
                    <div class="text-h6 mb-2">{{ item.label }}</div>
                    <div class="value-unit">
                    <span class="value">{{ weather[item.key] ?? '--' }}</span>
                    <span class="unit">{{ item.unit }}</span>
                    </div><div class="mt-2 text-caption grey--text" v-if="time[item.key]">
                    更新时间：{{ time[item.key] }}
                  </div>
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
import { ref, onMounted, computed, onUnmounted } from 'vue'
import { defineAsyncComponent } from 'vue'

const IconTemp = defineAsyncComponent(() => import('vue-material-design-icons/Thermometer.vue'))
const IconPressure = defineAsyncComponent(() => import('vue-material-design-icons/WeatherWindy.vue'))
const IconPM25 = defineAsyncComponent(() => import('vue-material-design-icons/WeatherDust.vue'))
const IconHumidity = defineAsyncComponent(() => import('vue-material-design-icons/WaterPercent.vue'))

const categories = [
  { key: 'temp', label: '温度', unit: 'C', icon: IconTemp },
  { key: 'pressure', label: '大气压强', unit: 'hPa', icon: IconPressure },
  { key: 'pm2_5', label: 'PM2.5', unit: 'μg/m³', icon: IconPM25 },
  { key: 'humidity', label: '湿度', unit: '%', icon: IconHumidity },
]

const weather = ref({})
const tempUnit = ref('C')
const error = ref(false)
const errorMsg = ref('')
const time = ref({})
const countdown = ref(30)
let timer = null
let interval = null

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

const refreshAll = () => {
  categories.forEach(cat => fetchWeather(cat.key))
}

onMounted(() => {
  refreshAll()
  timer = setInterval(() => {
    refreshAll()
    countdown.value = 30
  }, 30000)
  interval = setInterval(() => {
    if (countdown.value > 0) countdown.value--
  }, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
  clearInterval(interval)
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
  background: var(--md-background);
  color: var(--md-on-background);
}
.scrollable-container {
  overflow-y: auto;
  max-height: 100vh;
  width: 100%;
  box-sizing: border-box;
  background: var(--md-surface);
  color: var(--md-on-surface);
}
.text-h4 {
  font-size: 2.6rem;
  color: var(--md-primary);
  line-height: 1.1;
}
.value-unit {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  gap: 0.25em;
}
.value-unit .value {
  font-size: 2.6rem;
  font-weight: bold;
  color: var(--md-primary);
  line-height: 1.1;
}
.value-unit .temp-value {
  font-size: 2.4rem;
  font-weight: bold;
  color: var(--md-primary);
  line-height: 1.1;
}
.value-unit .unit {
  font-size: 1.2rem;
  font-weight: 500;
  color: var(--md-primary);
  margin-bottom: 0.2em;
}
.value-unit .temp-unit {
  font-size: 1.2rem;
  font-weight: 500;
  color: var(--md-primary);
  margin-left: 0.5em;
}
.v-sheet {
  background: var(--md-surface) !important;
  color: var(--md-on-surface) !important;
  border-radius: 4px;
  border: 1px solid var(--md-primary);
  box-shadow: 0 2px 8px 0 rgba(98,0,234,0.10), 0 1.5px 6px 0 rgba(0,0,0,0.10);
  transition: box-shadow 0.3s cubic-bezier(.25,.8,.25,1);
}
.card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: 160px;
  height: 100%;
}
.unit-switcher {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-top: 1px !important;
}
.card:hover {
  box-shadow: 0 6px 24px 0 rgba(98,0,234,0.18), 0 3px 12px 0 rgba(0,0,0,0.18);
}
.v-btn {
  color: var(--md-primary) !important;
}
.v-alert {
  background: var(--md-error) !important;
  color: var(--md-on-error) !important;
}
.v-alert .v-alert__icon,
.v-alert .v-alert__content {
  color: var(--md-on-error) !important;
}
.text-caption {
  color: var(--md-secondary) !important;
}
.v-card-title {
  color: var(--md-on-primary) !important;
  background: var(--md-primary) !important;
  height: 64px;
  min-height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  font-weight: 500;
  letter-spacing: 0.05em;
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
  padding: 0 24px;
  margin-bottom: 16px;
}
.temp-label {
  margin-bottom: 0 !important;
}
.no-margin-top {
  margin-top: 0 !important;
}
.weather-icon-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 8px;
}
.weather-icon {
  width: 36px;
  height: 36px;
  color: var(--md-primary);
}
</style>
