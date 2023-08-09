<!--//src/views/StartPage.vue-->

<script lang="ts">
import WeatherWidget from '@/components/weather-widget.vue';
import { markRaw, defineComponent } from 'vue';

interface Widget {
  component: any; // тип компонента
}

export default defineComponent({
  data() {
    return {
      widgets: [] as Widget[]
    };
  },
  mounted() {
    this.addWidget();
  },

  methods: {
    addWidget() {
      this.widgets.push({
        component: markRaw(WeatherWidget)
      });
    }
  }
});
</script>

<template>
  <button @click="addWidget">Добавить ещё один виджет</button>

  <hr />

  <div v-for="(widget, index) in widgets" :key="index">
    <component :is="widget.component"></component>
  </div>
</template>
