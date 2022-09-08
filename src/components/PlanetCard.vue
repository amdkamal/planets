<template>
  <div class="planet-card px-12 py-9 m-6 rounded-3xl">
    <div class="card-header font-gold text-xl mb-5 flex justify-between">
      <div>
        {{ this.timestamp(planet.created, "DD MMM yy") }}
      </div>
      <div class="hidden md:block">
        {{ this.timestamp(planet.created, "ddd MM, A") }}
      </div>
    </div>
    <div class="card-body capitalize flex justify-between items-center">
      <div class="flex items-center flex-row flex-wrap sm:flex-nowrap">
        <div
          class="card-image mr-6 overflow-hidden rounded-3xl md:w-20 sm:w-1/6 w-20 mb-4"
        >
          <img src="/icon.png" class="w-full" alt="" />
        </div>
        <div>
          <div class="font-bold font-white text-3xl mb-2">
            {{ planet.name }}
          </div>
          <div class="font-bold font-grey text-2xl mb-2 md:hidden">
            {{ planet.climate }}
          </div>
          <div class="font-grey text-xl">
            Appears in {{ planet.films.length }} films
          </div>
        </div>
      </div>

      <div class="hidden md:block">
        <div class="font-grey font-bold text-2xl">{{ planet.climate }}</div>
      </div>
    </div>
    <div class="expand-btn mr-9 cursor-pointer" @click="expandCard()">
      <img :src="isExpanded ? 'shrink.png' : 'expand.png'" width="55" alt="" />
    </div>
    <div :class="`card-expand ${isExpanded ? '' : 'hidden'}`">
      <div class="wrapper mt-4">
        <div
          v-for="film in planet.films"
          :key="film.url"
          class="font-grey text-xl font-bold"
        >
          {{ film.title }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import axios from "axios";
export default {
  props: ["planet"],
  data() {
    return {
      isExpanded: false,
    };
  },
  methods: {
    timestamp: (date, options) => {
      return moment(date).format(options);
    },
    expandCard: function () {
      this.isExpanded = !this.isExpanded;
    },
  },
  async mounted() {
    const response = await axios.get(`https://swapi.dev/api/films`);
    let films = response.data.results;
    let includedFilms = films.filter((film) => {
      return this.planet.films.includes(film.url);
    });
    this.planet.films = includedFilms;
  },
};
</script>

<style></style>
