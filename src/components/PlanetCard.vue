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
        <Loading v-if="isLoading" />
        <div v-else>
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
  </div>
</template>

<script>
import moment from "moment";
import axios from "axios";
import Loading from "./Loading.vue";
export default {
  props: ["planet"],
  components: { Loading },
  data() {
    return {
      isExpanded: false,
      isLoading: false,
      filmsList: [],
    };
  },
  methods: {
    timestamp: (date, options) => {
      return moment(date).format(options);
    },
    expandCard: async function () {
      if (this.isExpanded === true) {
        this.isExpanded = false;
      } else {
        this.isExpanded = true;
        try {
          this.isLoading = true;
          const response = await Promise.all(
            this.filmsList.map((film) => axios.get(film))
          ).then((values) => values.map((value) => value.data));
          this.planet.films = response;
        } catch (error) {
          console.log(error);
        }
      }
      this.isLoading = false;
    },
  },
  mounted() {
    this.filmsList = this.planet.films;
  },
};
</script>

<style></style>
