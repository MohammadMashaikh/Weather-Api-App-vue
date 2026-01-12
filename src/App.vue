<script setup>
import { ref } from 'vue';


const cards = ref([])



const API_KEY = "b5d795034043a04e1a36b146d942bd4e";
const city = ref('');

const header = ref([])
const hours = ref([])
const forecastByDay = ref({})
const selectedDay = ref(null)



  const fetchData = () => {
    // Empty input → do nothing
    if (!city.value.trim()) return

    const currentUrl =
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&appid=${API_KEY}`

    const forecastUrl =
      `https://api.openweathermap.org/data/2.5/forecast?q=${city.value}&units=metric&appid=${API_KEY}`

    // 1️⃣ FIRST: current weather
    fetch(currentUrl)
      .then(res => {
        if (!res.ok) throw new Error('City not found')
        return res.json()
      })
      .then(response => {
        // ✅ Update header safely
        header.value = [{
          cityName: response.name,
          temp: Math.round(response.main.temp),
          icon: response.weather[0].icon,
          main: response.weather[0].main,
          countryCode: response.sys.country,
          high: Math.round(response.main.temp_max),
          low: Math.round(response.main.temp_min),
        }]

        cards.value = [
        {
          label: 'Wind',
          value: `${Math.round(response.wind.speed)} km/h`,
          icon: '🌬️'
        },
        {
          label: 'Humidity',
          value: `${response.main.humidity}%`,
          icon: '💧'
        }
      ]


        // 2️⃣ ONLY NOW fetch forecast
        return fetch(forecastUrl)
      })
      .then(res => {
        if (!res.ok) throw new Error('Forecast error')
        return res.json()
      })
      .then(response => {
        // ✅ response.list is GUARANTEED to exist
        const grouped = {}

        response.list.forEach(item => {
          const date = item.dt_txt.split(' ')[0]

          if (!grouped[date]) grouped[date] = []

          grouped[date].push({
            time: formatHour(item.dt_txt),
            temp: Math.round(item.main.temp),
            icon: item.weather[0].icon,
            main: item.weather[0].main,
          })
        })

        forecastByDay.value = grouped

        selectedDay.value = Object.keys(grouped)[0]
        hours.value = grouped[selectedDay.value]
      })
      .catch(() => {
        // ✅ ONE clean error exit
        alert('City not found')

        // Optional cleanup
        header.value = []
        hours.value = []
        forecastByDay.value = {}
        cards.value = []
      })
  }



  const selectDay = (day) => {
    selectedDay.value = day
    hours.value = forecastByDay.value[day]
  }


  const formatHour = (dt) => {
    const date = new Date(dt)
    return date.toLocaleTimeString([], {
      hour: 'numeric',
      hour12: true
    })
  }

  const formatDay = (dateStr) => {
    return new Date(dateStr).toLocaleDateString([], {
      weekday: 'short'
    })
  }



</script>



<template>
  <!-- Background (ONLY ONCE) -->
  <div
  class="min-h-screen w-full
         bg-[radial-gradient(circle_at_top,_#9f7aea,_#4338ca_60%,_#312e81)]
         px-6 py-10"
>

  
    <!-- App Container -->
    <div
      :class="header.length > 0 ? 'min-h-screen' : 'h-auto'"
       class="relative mx-auto w-full max-w-7xl
            rounded-[36px]
            bg-gradient-to-br from-white/10 to-white/5
            backdrop-blur-2xl
            border border-white/20
            shadow-[0_60px_120px_rgba(0,0,0,.35)]
            overflow-hidden
            transition-all duration-500"
    >



    <!-- 🔍 Search -->
    <div class="relative z-10 p-8">
      <form
        @submit.prevent="fetchData"
        class="flex items-center gap-4
              rounded-3xl
              bg-gradient-to-br from-white/20 to-white/5
              backdrop-blur-2xl
              px-6 py-5
              shadow-[0_18px_45px_rgba(0,0,0,.35),_inset_0_1px_1px_rgba(255,255,255,.4)]
              border border-white/20"
      >
        <!-- Icon -->
        <span class="text-white/70 text-xl select-none">🔍</span>

        <!-- Input -->
        <input
          v-model="city"
          type="text"
          placeholder="Search for a city..."
          class="w-full bg-transparent outline-none
                text-white text-lg
                placeholder-white/50"
        />

        <!-- Submit Button -->
        <button
          type="submit"
          class="rounded-xl px-5 py-2
                bg-white/20 text-white font-medium
                shadow-[inset_0_1px_1px_rgba(255,255,255,.5)]
                hover:bg-white/30 transition"
        >
          Search
        </button>
      </form>
    </div>



      <!-- Ambient Glow -->
      <div class="absolute -top-32 -right-32 w-80 h-80 bg-purple-500/30 rounded-full blur-3xl"></div>

      <!-- Center Hero Weather -->
      <div
      v-for="head in header" :key="head.cityName"
        class="relative z-10 px-8 pt-6 pb-2
              flex flex-col items-center text-center text-white"
      >
        <!-- City -->
        <p class="text-2xl font-semibold tracking-wide opacity-90">
          {{ head.cityName }} | {{ head.countryCode }}
        </p>

        <!-- Icon -->
        <div class="text-[96px] leading-none drop-shadow-[0_20px_40px_rgba(0,0,0,.4)]">
          <img :src="`https://openweathermap.org/img/wn/${head.icon}@4x.png`" alt="">
          
        </div>

        <!-- Temperature -->
        <p class="text-8xl font-bold tracking-tight -mt-2">
          {{ head.temp }}°
        </p>

        <!-- Description -->
        <p class="text-base opacity-70 mt-1">
          {{ head.main }}
        </p>

        <!-- High / Low -->
        <p class="text-sm opacity-60 mt-2">
          H:{{ head.high }}°  L:{{ head.low }}°
        </p>
      </div>


      <!-- Days Selector -->
      <div
        v-if="Object.keys(forecastByDay).length > 0"
        class="relative z-10 px-8 mt-6 flex gap-3 overflow-x-auto"
      >
        <button
          v-for="day in Object.keys(forecastByDay)"
          :key="day"
          @click="selectDay(day)"
          class="px-5 py-2 rounded-xl text-sm font-medium
                transition
                backdrop-blur-xl
                border border-white/20
                shadow-[inset_0_1px_1px_rgba(255,255,255,.4)]"
          :class="day === selectedDay
            ? 'bg-white/30 text-white'
            : 'bg-white/10 text-white/70 hover:bg-white/20'"
        >
          {{ formatDay(day) }}
        </button>
      </div>


      <!-- Hourly Forecasts -->
      <div v-if="header.length > 0"
        class="relative z-10 mx-8 mt-4 rounded-[32px]
               bg-gradient-to-br from-white/25 to-white/5
               backdrop-blur-2xl
               shadow-[0_20px_60px_rgba(0,0,0,.35),_inset_0_1px_1px_rgba(255,255,255,.4)]
               p-6"
      >
        <p class="text-white/70 text-sm mb-4">Hourly Forecast</p>

        <div class="flex gap-4 overflow-x-auto pb-2">
          <div
            v-for="h in hours"
            :key="h.time"
            class="min-w-[72px] rounded-2xl bg-white/15
                   backdrop-blur-xl
                   shadow-[inset_0_1px_1px_rgba(255,255,255,.4)]
                   text-white flex flex-col items-center py-4"
          >
            <span class="text-xs opacity-70">{{ h.time }}</span>
            <span class="text-xl my-2"><img :src="`https://openweathermap.org/img/wn/${h.icon}@2x.png`" alt=""></span>
            <span class="text-sm font-medium">{{ h.temp }}°</span>
            <span class="text-[10px] opacity-60 mt-1">{{ h.main }}</span>

          </div>
        </div>
      </div>


      <!-- Floating Weather Cards -->
      <div class="relative z-10 mt-8 grid grid-cols-1 md:grid-cols-2 gap-6 px-8 pb-10">
        <div
          v-for="card in cards"
          :key="card.label"
          class="rounded-[28px] p-6
                 bg-gradient-to-br from-white/20 to-white/5
                 backdrop-blur-2xl
                 shadow-[0_18px_45px_rgba(0,0,0,.35),_inset_0_1px_1px_rgba(255,255,255,.35)]
                 text-white flex justify-between items-center"
        >
          <div>
            <p class="text-sm opacity-60">{{ card.label }}</p>
            <p class="text-3xl font-semibold mt-1">{{ card.value }}</p>
          </div>

          <div class="text-5xl drop-shadow-lg">
            {{ card.icon }}
          </div>
        </div>
      </div>

    </div>
  </div>
</template>
