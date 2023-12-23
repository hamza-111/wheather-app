<script setup>
const config = useRuntimeConfig()
const cookie = useCookie("city")
if (!cookie.value) {
  cookie.value ="Gabes"
}
const cityName = ref(cookie.value)
const cityNameForm = ref("")
const bkg = ref(null)
// const { data: city, error } = useFetch(
//   () => `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(cityName.value)}&units=metric&appid=${key}`
// );

const { data: city, error } = useAsyncData("city", async () => {
    const response = await $fetch(`https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(cityName.value)}`, {
      params: {
        units : "metric",
        appid : config.public.KEY,

      }
    });
    const tmp = response.main.temp;
    cookie.value = cityName.value;
    if (tmp <= -10) {
      bkg.value = "https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
    } else if (tmp > -10 && tmp < 0) {
      bkg.value = "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80"
    } else if (tmp > 0 && tmp <= 10) {
      bkg.value = "https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80"
    } else {
      bkg.value = "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80"
    }
  return response
},{
  watch : [cityName]
});

const todays = new Date().toLocaleDateString("en-US", { weekday: "long", year: "numeric", month: "long", day: "numeric" });
const inputCity = () => {
  cityName.value = cityNameForm.value.trim();
  cityNameForm.value = "";
}
</script>

<template>
  <div class="h-screen relative">
    <img :src="bkg" />
    <div class="absolute h-full w-full top-0 overlay"></div>
    <div class="absolute h-full w-full top-0 p-48">
      <div class="flex justify-between">
        <div>
          <!-- Vérification si 'city' est défini avant d'accéder à ses propriétés -->
          <template v-if="error">
            <p class="text-white">Une erreur s'est produite lors de la récupération des données météorologiques.</p>
          </template>
          <template v-else>
            <h1 class="text-white text-7xl">{{ city && city.name ? city.name : 'Unknown City' }}</h1>
            <p class="font-extralight text-2xl mt-2 text-white">
              {{todays}}
            </p>
            <!-- Vérification de la présence de 'city.weather' -->
            <img v-if="city && city.weather && city.weather[0]" class="w-56 icon" :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"/>
          </template>
        </div>
        <div>
          <!-- Vérification si 'city.main' est défini avant d'accéder à 'temp' -->
          <p v-if="city && city.main" class="text-9xl text-white font-extralight">{{ city.main.temp }}°</p>
        </div>
      </div>
      <div class="mt-20">
        <input class="w-1/2 h-10" type="text" placeholder="search a city..." v-model="cityNameForm" @keyup.enter="inputCity" />
        <button @click="inputCity" class="h-10 w-20 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4">
          Search
        </button>
      </div>
    </div>
  </div>
</template>

<style scope>
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}

.icon {
  margin-left: -2.75rem;
  margin-right: -2.75rem;
}
</style>
