<script setup>
  import { ref, watch, computed } from 'vue'
  import { cities } from './cities.js'
  import WeatherEffect from './components/WeatherEffect.vue'

  const apiKey = "c37a596c8ab43dc652ecbc768b9e8e1d"
  const selectedCity = ref("")
  const weatherData = ref(null)
  const loading = ref(false)
  const errorMsg = ref("")
  const localTime = ref("")
  const isNight = ref(false)

  const updateLocalTime = () => {
    if(!weatherData.value) return

    const dt = weatherData.value.dt
    const timezone = weatherData.value.timezone
    const sunrise = weatherData.value.sys.sunrise
    const sunset = weatherData.value.sys.sunset

    const localDate = new Date((dt + timezone) * 1000)
    const hours = localDate.getUTCHours().toString().padStart(2, "0")
    const minutes = localDate.getUTCMinutes().toString().padStart(2, "0")
    localTime.value = `${hours}: ${minutes}`

    const now = dt + timezone
    isNight.value = now < sunrise + timezone || now > sunset + timezone
  }

  const selectedCityName = computed(() =>{
    const cityObj = cities.find(c => c.query === selectedCity.value)
    return cityObj ? cityObj.name : ""
  })

  watch(selectedCity, async(city) =>{
    if(!city) return;
    await fetchWeather(city)
  })

  async function fetchWeather(city){
    loading.value = true
    weatherData.value =null
    errorMsg.value = ""

    try{
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${city},tw&appid=${apiKey}&units=metric&lang=zh_tw`
      const res = await fetch(url)

      if(!res.ok){
        throw new Error("查詢失敗")
      }
      const data = await res.json()
      weatherData.value = data
      updateLocalTime()
    }catch(err){
      errorMsg.value = err.message
    }finally{
      loading.value = false
    }
    console.log(weatherData.value)
  }
</script>

<template>
  <div :class="['weatherapp', isNight? 'nightbackground': 'daybackground']">
    <div :class="['container', isNight ? 'night' : 'day']" >
      <WeatherEffect v-if="weatherData" :weatherMain="weatherData.weather[0].main" />
        <p class="loading" v-if="loading">查詢中...</p>
        <p class="errorMsg" v-if="errorMsg">{{ errorMsg }}</p>
      <div class="title">
        <select name="Location" id="" v-model="selectedCity">
          <option value="">選擇城市</option>
          <option v-for="city in cities" :key="city.query" :value="city.query">{{ city.name }}</option>
        </select>
        <h1 v-if="weatherData">{{ selectedCityName }}天氣</h1>
      </div>

      <div class="content">
        <div class="temp" v-if="weatherData">
          <p>氣溫</p>
          <span>{{ weatherData.main.temp }}°C</span>
          <p>狀態</p>
          <span>{{ weatherData.weather[0].description }}</span>
          <p>體感溫度</p>
          <span>{{ weatherData.main.feels_like }}°C</span>
          <p>今日最高/低溫</p>
          <span>{{ weatherData.main.temp_min }}°C ~ {{ weatherData.main.temp_max }}°C</span>
          <div class="image">
            <img 
            :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" 
            :alt="weatherData.weather[0].description" />
          </div>
        </div>

        <div class="humidityandPressure" v-if="weatherData && weatherData.main">
          <p>濕度</p>
          <span>{{ weatherData.main.humidity }}%</span>
          <p>氣壓</p>
          <span>{{ weatherData.main.pressure }}hPa</span>
        </div>

        <div class="wind" v-if="weatherData && weatherData.wind">
          <p>風速</p>
          <span>{{ weatherData.wind.speed }}m/s</span>
          <p>風向</p>
          <span>{{ weatherData.wind.deg }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>  
  .weatherapp {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100vh;
    background: linear-gradient(90deg,rgba(74, 134, 150, 1) 1%, rgba(74, 143, 121, 1) 32%, rgba(57, 133, 90, 1) 53%, rgba(150, 142, 51, 1) 100%);
  }

  .nightbackground {
    background: linear-gradient(0deg,rgba(40, 62, 77, 1) 0%, rgba(33, 58, 79, 1) 19%, rgba(33, 45, 105, 1) 39%, rgba(27, 23, 110, 1) 73%, rgba(35, 11, 64, 1) 100%);
  }

  .daybackground {
    background: linear-gradient(0deg,rgba(186, 178, 138, 1) 9%, rgba(155, 177, 186, 1) 38%, rgba(113, 153, 168, 1) 53%, rgba(91, 136, 171, 1) 83%, rgba(76, 151, 161, 1) 100%);
  }

  .container {
    position: absolute;
    background: linear-gradient(90deg,rgba(111, 214, 242, 1) 1%, rgba(119, 230, 195, 1) 32%, rgba(99, 230, 155, 1) 53%, rgba(242, 229, 80, 1) 100%);
    margin: auto;
    padding: 20px;
    top: 100px;
    bottom: 100px;
    right: 100px;
    left: 100px;
    border-radius: 10px;
    box-shadow: 0 0 100px rgba(0,0,0,0.5);
  }

  .day {
    background: linear-gradient(0deg,rgba(235, 224, 171, 1) 9%, rgba(194, 224, 237, 1) 38%, rgba(155, 212, 235, 1) 53%, rgba(124, 185, 242, 1) 81%, rgba(109, 222, 237, 1) 100%);
    color: black;
  }
  
  .night {
    background: linear-gradient(0deg,rgba(64, 100, 122, 1) 0%, rgba(53, 94, 130, 1) 19%, rgba(49, 67, 158, 1) 39%, rgba(36, 30, 150, 1) 73%, rgba(46, 15, 84, 1) 100%);
    color: white;
  }

  .title {
    display: flex;
    align-items: center;
    margin: 0;
  }

  .container h1 {
    width: 100%;
    display: flex;
    justify-content: center;
    margin: 0 0 10px 0;
  }

  .container .loading, .errorMsg {
    position: fixed;
    bottom: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 36px;
    font-weight: 600;
  }

  select {
    background: transparent;
    border: 2px solid #999;
    border-radius: 10px;
    font-size: clamp(18px, 4vw, 28px);
    font-weight: 600;
    color: #fff;
    outline: none;
    padding: 6px;
    position: fixed;
    cursor: pointer;
  }

  option {
    background: #555;
  }

  .content {
    width: 100%;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat((1, 1fr));
  }

  .content div {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
  }

  .content p {
    font-size: clamp(20px, 3vw, 32px);
    font-weight: 600;
    margin: 0;
    padding: 0;
  }

  .content span {
    font-size: clamp(16px, 2.5vw, 24px);
  }

  .temp {
    /* background: red; */
    display: flex;
    flex-direction: column;
  }

  img {
    max-width: 100%;
    height: auto;
    margin: 0 auto;
  }
  /* 平板 */
  @media (max-width: 768px) {
    .weatherapp {
      margin: 0;
      padding: 0;
    }

    .container {
      width: 100%;
      height: 100%;
      margin: 0;
      padding: 0;
      top: 0;
      left: 0;
    }

    select {
      position: relative;
      font-size: 20px;
      margin: 0 auto;
      border: none;
      appearance: none;
      left: 10px;
    }

    .container h1 {
      display: none;
    }
  
    .content {
    grid-template-columns: 1fr;
    }

    .image {
      position: absolute;
      bottom: 0;
      width: 100%;
      height: auto;
    }
  }
</style>
