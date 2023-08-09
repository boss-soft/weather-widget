<!--//src/components/weather-widget.vue-->

<script lang="ts">
interface WeatherInterface {
  coord: {
    lon: number;
    lat: number;
  };
  weather: [
    {
      id: number;
      main: string;
      description: string;
      icon: string;
    }
  ];
  base: string;
  main: {
    temp: number;
    feels_like: number;
    temp_min: number;
    temp_max: number;
    pressure: number;
    humidity: number;
  };
  visibility: number;
  wind: {
    speed: number;
    deg: number;
  };
  clouds: {
    all: number;
  };
  dt: number;
  sys: {
    type: number;
    id: number;
    country: string;
    sunrise: number;
    sunset: number;
  };
  timezone: number;
  id: number;
  name: string;
  cod: number;
}

interface RangeInterface {
  min: number;
  max: number;
  value: string;
}

export default {
  name: 'weather-widget',

  data() {
    return {
      result: '',

      cities: ['London, UK', 'Moscow, RU', 'New York, US'],
      currentCity: 0,

      iconUrl: '',
      temperature: 'C',
      feeling: '',
      wind: '',
      pressure: '',
      humidity: '',
      visibility: '',
      dew: '',

      readyToShow: false,
      showOption: false,
      cityError: null as number | null,

      editLocation: '',

      dragIndex: null as number | null,
      dragTargetIndex: null as number | null
    };
  },

  mounted() {
    this.fetchWeather();
  },

  methods: {
    async fetchWeather() {
      this.readyToShow = false;
      const response = await fetch(
        'https://api.openweathermap.org/data/2.5/weather?q=' +
          this.cities[this.currentCity] +
          '&appid=179a244e375f01e7d509774314dd69c7&units=metric'
      );
      if (!response.ok) {
        this.cityError = this.currentCity;
      }
      const data = await response.json();
      if (data != undefined) this.fillingTheFields(data);
    },

    getNameByValue: function (RANGES: RangeInterface[], value: number): string {
      let name: string = '';
      switch (true) {
        default:
          RANGES.forEach((elem: RangeInterface) => {
            if (value >= elem.min && value < elem.max) {
              name = elem.value as string;
            }
          });
          break;
      }
      return name;
    },

    windDirection(grad: string | number) {
      grad = +grad;
      if (isNaN(grad)) return '';

      const RANGES: RangeInterface[] = [
        { min: 337.5, max: 22.4, value: 'N' },
        { min: 22.5, max: 67.4, value: 'NW' },
        { min: 67.5, max: 112.4, value: 'W' },
        { min: 112.5, max: 157.4, value: 'WS' },
        { min: 157.5, max: 202.4, value: 'S' },
        { min: 202.5, max: 247.4, value: 'SE' },
        { min: 247.5, max: 292.4, value: 'E' },
        { min: 292.5, max: 337.4, value: 'EN' }
      ];

      return this.getNameByValue(RANGES, grad);
    },

    cloudy(percent: string | number) {
      percent = +percent;
      if (isNaN(percent)) return '';

      const RANGES: RangeInterface[] = [
        { min: 0, max: 9, value: 'Clear' },
        { min: 10, max: 29, value: 'Party cloudy' },
        { min: 30, max: 49, value: 'Party sunny' },
        { min: 50, max: 89, value: 'Cloudy' },
        { min: 90, max: 100, value: 'Overcast' }
      ];

      return this.getNameByValue(RANGES, percent);
    },

    windiness(speed: string | number) {
      speed = +speed;
      if (isNaN(speed)) return '';

      const RANGES: RangeInterface[] = [
        { min: 0.0, max: 0.2, value: 'Calm' },
        { min: 0.3, max: 1.5, value: 'Light air' },
        { min: 1.6, max: 3.3, value: 'Light breeze' },
        { min: 3.4, max: 5.4, value: 'Gentle breeze' },
        { min: 5.5, max: 7.9, value: 'Moderate breeze' },
        { min: 8.0, max: 10.7, value: 'Fresh breeze' },
        { min: 10.8, max: 13.8, value: 'Strong breeze' },
        { min: 13.9, max: 17.1, value: 'Moderate gale' },
        { min: 17.2, max: 20.7, value: 'Fresh gale' },
        { min: 20.8, max: 24.4, value: 'Strong gale' },
        { min: 24.5, max: 28.4, value: 'Whole gale' },
        { min: 28.5, max: 32.6, value: 'Storm' },
        { min: 32.7, max: 99999.9, value: 'Hurricane' }
      ];
      return this.getNameByValue(RANGES, speed);
    },

    fillingTheFields(result: WeatherInterface) {
      if ('message' in result) {
        this.showOption = true;
        this.cityError = this.currentCity;
        return;
      }

      this.iconUrl = `https://openweathermap.org/img/wn/${result['weather'][0]['icon']}@2x.png`;
      this.temperature = this.temperatureRounding(result['main']['temp']) + '°C';
      this.feeling = `Feels like ${this.temperatureRounding(result['main']['feels_like'])}°C. `;
      this.feeling += `Clouds ${this.cloudy(result['clouds']['all'])}. `;
      this.feeling += this.windiness(result['wind']['speed']);
      this.wind = `${result['wind']['speed']}m/s ${this.windDirection(result['wind']['deg'])}`;
      this.pressure = result['main']['pressure'] + 'hPa';
      this.humidity = `Humidity: ${result['main']['humidity']}%`;
      this.visibility = `Visibility: ${this.metersToKm(result['visibility'])}km`;
      this.dew = `Dew point: ${this.dewPoint(
        result['main']['temp'],
        result['main']['humidity']
      )}°C`;
      this.cityError = null;
      this.showOption = false;
      this.readyToShow = true;
    },

    dewPoint(temperature: number, humidity: number) {
      const f = (17.27 * temperature) / (237.7 + temperature) + Math.log(humidity / 100);
      return Math.round((237.7 * f) / (17.27 - f));
    },

    temperatureRounding(grad: string | number): string {
      grad = +grad;
      if (isNaN(grad)) return '';

      return Math.round(grad).toString();
    },
    metersToKm(meters: string | number): string {
      meters = +meters;
      if (isNaN(meters)) return '';
      return (meters / 1000).toFixed(1);
    },
    startDrag(index: number) {
      this.dragIndex = index;
    },

    dragOver(index: number) {
      this.dragTargetIndex = index;
    },

    endDrag() {
      if (this.dragIndex != null && this.dragTargetIndex != null) {
        if (this.currentCity == this.dragIndex) {
          this.currentCity = this.dragTargetIndex;
        } else if (this.currentCity == this.dragTargetIndex) {
          this.currentCity = this.dragIndex;
        }

        const dragItem = this.cities[this.dragIndex];
        this.cities[this.dragIndex] = this.cities[this.dragTargetIndex];
        this.cities[this.dragTargetIndex] = dragItem;

        this.dragIndex = null;
        this.dragTargetIndex = null;
      }
    },

    setCity(index: number): void {
      this.currentCity = index;
      this.fetchWeather();
    },

    delCity(index: number): void {
      if (index == this.currentCity) {
        this.currentCity = 0;
        this.cityError = null;
      }
      this.cities.splice(index, 1);

      if (this.currentCity >= this.cities.length) {
        this.currentCity = 0;
      }
    },

    addCity(): void {
      this.cities.push(this.editLocation);
      this.editLocation = '';
    }
  }
};
</script>

<template>
  <div class="container">
    <template v-if="showOption">
      <div class="w100">
        <div class="row">
          <div class="column">
            <strong> Settings </strong>
          </div>
          <div class="right-column">
            <img
              alt="settings"
              class="icon fake-link"
              src="../assets/cross-svgrepo-com.png"
              @click="fetchWeather()"
            />
          </div>
        </div>

        <div
          v-for="(city, index) in cities"
          :key="index"
          :class="{ 'text-red': cityError == index, bold: currentCity == index }"
          class="row elements"
        >
          <div class="column">
            <img
              alt="draggable"
              class="icon draggable"
              draggable="true"
              src="../assets/line-horizontal-3-svgrepo-com.png"
              @dragend="endDrag()"
              @dragenter="dragOver(index)"
              @dragstart="startDrag(index)"
              @dragover.prevent
            />
            <span class="fake-link" @click="setCity(index)">
              {{ city }}
            </span>
          </div>
          <div class="right-column">
            <img
              alt="trash"
              class="icon fake-link"
              src="../assets/trash-can-svgrepo-com.png"
              @click="delCity(index)"
            />
          </div>
        </div>

        <div v-if="cityError" class="row">
          <h4 class="text-red">Такого города не существует</h4>
        </div>

        <div class="input-group">
          <div class="w100 text-left">Add location:</div>
          <div class="row">
            <input v-model="editLocation" class="w85" placeholder="New York" type="text" />
            <img
              alt="enter"
              class="icon fake-link"
              src="../assets/enter-svgrepo-com.png"
              @click="addCity()"
            />
          </div>
        </div>
      </div>
    </template>

    <template v-else-if="readyToShow">
      <div class="row">
        <div class="column">
          <strong>
            {{ cities[currentCity] }}
          </strong>
        </div>
        <div class="right-column">
          <img
            alt="settings"
            class="icon fake-link"
            src="../assets/gear-svgrepo-com.png"
            @click="showOption = true"
          />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <img :src="iconUrl" alt="clouds" class="clouds" />
        </div>
        <div class="column">
          <span class="temperature">
            {{ temperature }}
          </span>
        </div>
      </div>

      <div class="row">
        {{ feeling }}
      </div>

      <div class="row">
        <div class="column">
          <img alt="wind" class="icon" src="../assets/wind-duotone-svgrepo-com.png" />
          {{ wind }}
        </div>
        <div class="column">
          <img alt="pressure" class="icon" src="../assets/gauge-high-svgrepo-com.png" />
          {{ pressure }}
        </div>
      </div>

      <div class="row h-center">
        <div class="column">
          {{ humidity }}
        </div>
        <div class="column">
          {{ dew }}
        </div>
      </div>

      <div class="row">
        {{ visibility }}
      </div>
    </template>
    <template v-else>
      <div class="h-center">
        <div class="loader">
          <div class="wheel"></div>
        </div>
      </div>
    </template>
  </div>
</template>

<style scoped>
.container {
  width: 250px;
  padding: 10px;
  box-shadow: 3px 3px 15px rgba(0, 0, 0, 0.5);
  font-family: sans-serif;
  font-size: 13px;
  border-radius: 10px;
  margin: 20px;
  align-items: center;
  justify-content: center;
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  width: 100%;
}

.h-center {
  width: 100%;
  align-items: center;
  justify-content: center;
  display: flex;
}

.elements {
  background: #a9a9a994;
  padding: 5px 0;
}

.w100 {
  width: 100%;
}

.text-left {
  text-align: left;
}

.w85 {
  width: 85%;
}

.column {
  display: flex;
  flex: 1;
  align-items: center;
}

.right-column {
  text-align: right;
}

.icon {
  width: 20px;
}

.temperature {
  font-size: 33px;
  font-weight: bold;
}

.bold {
  font-weight: bold;
}

.loader {
  height: 60px;
  width: 60px;
}

.wheel {
  height: 60px;
  width: 60px;
  border: 6px solid transparent;
  border-top: 6px solid #555;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: auto;
}

.fake-link {
  cursor: pointer;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.text-red {
  color: red;
}

.draggable {
  cursor: move;
  position: relative;
}
</style>
