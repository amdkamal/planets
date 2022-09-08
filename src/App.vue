<template>
  <main>
    <ul>
      <li v-for="planet in planets" :key="planet.id">
        <PlanetCard :planet="planet" />
      </li>
    </ul>
  </main>
</template>

<script>
import axios from "axios";
import PlanetCard from "./components/PlanetCard.vue";

export default {
  data() {
    return {
      planets: [],
    };
  },
  methods: {
    async oneFunctionToGetThemAll() {
      //function to fetch all results through pages
      const APIRequest = async (url, data) => {
        data = data || [];
        await axios.get(url).then((res) => {
          data.push(...res.data.results);
          if (res.data.next !== null) {
            return APIRequest(res.data.next, data);
          }
        });
        return data;
      };

      //all planets
      const allPlanets = await APIRequest("https://swapi.dev/api/planets");
      //planets that have at least 2 film
      const planetsInFilms = allPlanets.filter(
        (planet) => planet.films.length > 1
      );

      //all people
      const people = await APIRequest("https://swapi.dev/api/people");

      //planets that appear in at least 2 movies and have residents with min total of 5 starships
      let planets = planetsInFilms;
      planets.forEach((planet) => {
        let newResidentList = people.filter((person) =>
          planet.residents.includes(person.url)
        );
        planet.residents = newResidentList;
      });
      planets = planets.filter((planet) => {
        let totalStarships = 0;
        planet.residents.forEach(
          (resident) => (totalStarships += resident.starships.length)
        );

        return totalStarships >= 5;
      });
      this.planets = planets;
    },
  },
  mounted() {
    this.oneFunctionToGetThemAll();
  },
  components: { PlanetCard },
};
</script>

<style>
@import url("./index.css");
</style>
