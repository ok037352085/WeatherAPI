<script setup>
    import { computed } from 'vue'

    const props = defineProps({
        weatherMain : String //接收OpenWeatherAPI 回傳的 weather[0].main
    })

    const effect = computed(() => {
        if(!props) return null
        const main = props.weatherMain.toLocaleLowerCase()

        if(main.includes('rain') || main.includes('drizzle')) return 'rain'
        if(main.includes('thunderstorm')) return 'thunder'
        if(main.includes('snow')) return 'snow'
        if(main.includes('mist') || main.includes('fog')) return 'fog'
        if(main.includes('cloud')) return 'cloud'
        if(main.includes('clear')) return 'clear'
        return null
    })
</script>

<template>
    <div class="weather-effect">
        <!-- rain -->
         <template v-if="effect === 'rain'">
            <div class="rain" v-for="n in 99" :key="n" :style="{left: `${Math.random() * 100}%`, animationDuration: `${Math.random() * 2 + 1}s`}"></div>
         </template>
         <!-- 雷雨 -->
        <template v-else-if="effect === 'thunder'">
            <div class="rain" v-for="n in 60" :key="n" :style="{left: `${Math.random() * 100}%`, animationDuration: `${Math.random() * 2 + 1}s`}"></div>
            <div class="lightning"></div>
         </template>
         <!-- snow -->
          <template v-else-if="effect === 'snow'">
            <div class="snow" v-for="n in 15" :key="n" :style="{left: `${Math.random() * 100}%`, animationDuration: `${Math.random() * 3 + 2}s`}"></div>
          </template>
         <!-- 起霧 -->
          <div class="fog" v-else-if="effect === 'fog'"></div>
          <!-- cloud -->
           <div class="clouds" v-else-if="effect === 'cloud'"></div>
           <!-- clear -->
            <div class="sun" v-else-if="effect === 'clear'"></div>
    </div>
</template>

<style scoped>
    .weather-effect {
        position: absolute;
        inset: 0;
        pointer-events: none;
        overflow: hidden;
    }

    .rain {
        position: absolute;
        bottom: 0;
        width: 2px;
        height: 20px;
        background: rgba(0,0,0,0.5);
        border-radius: 50%;
        animation: rain-drop linear infinite;
    }

    @keyframes rain-drop {
        0%{ transform: translateY(-100vh); opacity: 1; }
        100%{ transform: translateY(100vh); opacity: 0; }
    }

    .snow {
        position: absolute;
        top: 0;
        width: 8px;
        height: 8px;
        background: #fff;
        border-radius: 50%;
        animation: snow-fall linear infinite;
    }

    @keyframes snow-fall {
        0%{ transform: translateY(-10vh); opacity: 1; }
        100%{ transform: translateY(110vh); opacity: 0; }        
    }

    .fog {
        position: absolute;
        inset: 0;
        background: rgba(255,255,255,0.3);
        backdrop-filter: blur(8px);
    }

    .sun {
        position: absolute;
        top: 20px;
        right: 20px;
        width: 80px;
        height: 80px;
        background: radial-gradient(circle, yellow, orange);
        border-radius: 50%;
        box-shadow: 0 0 40px yellow;
    }

    .lightning {
        position: absolute;
        top: 0;
        left: 50%;
        width: 4px;
        height: 100%;
        background: white;
        opacity: 0;
        animation: flash 3s infinite;
    }

    @keyframes flash {
        0%, 95%, 100% { opacity: 0; }
        96% { opacity: 1; }
        97% { opacity: 0; }
        98% { opacity: 1; }
    }
</style>