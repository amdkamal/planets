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
    <div class="card-body capitalize flex md:justify-between items-center">
      <div
        class="flex items-center sm:flex-nowrap flex-row-reverse md:flex-row justify-between"
      >
        <div
          class="card-image mr-6 overflow-hidden rounded-3xl md:w-20 sm:w-1/6 w-1/5 mb-4"
        >
          <img src="/icon.png" class="w-full" alt="" />
        </div>
        <div class="flex items-center">
          <div
            class="block md:hidden text-3xl font-white font-bold bg-grey w-20 h-20 mr-4 rounded-full flex justify-center items-center"
          >
            <p>{{ planet.name.charAt(0) }}</p>
          </div>
          <div>
            <div class="font-bold font-white text-3xl mb-2">
              {{ planet.name }}
            </div>
            <div
              class="font-regular md:font-bold font-grey text-2xl mb-2 md:hidden"
            >
              {{ planet.climate }}
            </div>
            <div class="planet-films text-xl">
              Appears in {{ planet.films.length }} films
            </div>
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
        <div
          v-else
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
import Loading from "./Loading.vue";
export default {
  props: ["planet"],
  components: { Loading },
  data() {
    return {
      isExpanded: false,
      isLoading: false,
      filmsList: [],
      filmsInStorage: null,
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
        this.isLoading = true;
        try {
          const requestArr = this.filmsList.map(async (film) => {
            return axios.get(film);
          });
          if (!this.filmsInStorage.length) {
            //get films and populate storage
            const returnedFilms = await Promise.all(requestArr).then(
              (values) => {
                return values.map((value) => value.data);
              }
            );
            this.planet.films = returnedFilms;
            this.filmsInStorage = returnedFilms;
          } else {
            let existingFilms = this.filmsInStorage.filter((film) => {
              return this.filmsList.includes(film.url);
            });
            let nonExistingFilms = this.filmsInStorage.filter((film) => {
              return !this.filmsList.includes(film.url);
            });

            if (!nonExistingFilms.length) {
              //get films from storage if they all exist
              this.planet.films = existingFilms;
            } else {
              //only fetch non existing films then add them to storage
              let newRequests = nonExistingFilms.map((film) =>
                axios.get(film.url)
              );
              let newData = await Promise.all(newRequests).then((values) => {
                return values.map((value) => value.data);
              });
              this.planet.films = [newData, existingFilms];
              this.filmsInStorage.push(...newData);
            }
          }
        } catch (error) {
          console.log(error);
        }
        this.isLoading = false;
      }
    },
  },
  mounted() {
    this.filmsList = this.planet.films;
    this.filmsInStorage = JSON.parse(localStorage.getItem("films")) || [];
  },
  watch: {
    filmsInStorage(newValue, oldValue) {
      //updates local storage when state changes
      localStorage.setItem("films", JSON.stringify(newValue));
    },
  },
};
</script>

<style></style>
