<script>
import axios from "axios";

export default {
  data() {
    return {
      searchText: "",
      selectedContinent: "",
      countries: [],
      filteredCountries: [],
      selectedCountry: null,
      continents: ["Asia", "Africa", "Europe", "Oceania"],
      showDetailsPage: false,
      darkMode: false,
    };
  },
  computed: {
    filteredCountries() {
      return this.countries.filter((country) => {
        const nameMatch = country.name.common
          .toLowerCase()
          .includes(this.searchText.toLowerCase());
        const continentMatch = this.selectedContinent
          ? country.region === this.selectedContinent
          : true;
        return nameMatch && continentMatch;
      });
    },
    continents() {
      const uniqueContinents = new Set();
      this.countries.forEach((country) => uniqueContinents.add(country.region));
      return Array.from(uniqueContinents);
    },
  },
  mounted() {
    this.fetchCountries();
  },
  methods: {
    fetchCountries() {
      axios
        .get("https://restcountries.com/v3.1/all")
        .then((response) => {
          this.countries = response.data;
          this.filteredCountries = response.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    filterCountries() {
      if (this.searchText) {
        this.filteredCountries = this.countries.filter((country) =>
          country.name.common.includes(this.searchText)
        );
      } else if (this.selectedContinent) {
        this.filteredCountries = this.countries.filter(
          (country) => country.region === this.selectedContinent
        );
      } else {
        this.filteredCountries = this.countries;
      }
    },
    showDetails(country) {
      this.selectedCountry = country;
      this.showDetailsPage = true;
    },
    goBack() {
      this.selectedCountry = null;
      this.showDetailsPage = false;
    },
    showBorderDetails(border) {
      const borderCountry = this.countries.find(
        (country) => country.cca3 === border
      );
      this.selectedCountry = borderCountry;
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode;

      localStorage.setItem("darkMode", JSON.stringify(this.darkMode));
      if (this.darkMode) {
        document.body.classList.add("dark-mode");
      } else {
        document.body.classList.remove("dark-mode");
      }
    },
  },
  created() {
    const darkModePreference = localStorage.getItem("darkMode");
    if (darkModePreference) {
      this.darkMode = JSON.parse(darkModePreference);
    }
    if (this.darkMode) {
      document.body.classList.add("dark-mode");
    }
  },
};
</script>
<template>
  <div class="flex">
    <h2>where in the world ?</h2>
    <div @click="toggleDarkMode">
      <img src="./icon-moon.svg" style="background-color: black" />
      Dark Mode
    </div>
  </div>
  <div style="margin-bottom: 50px; margin-left: 40px">
    <input
      style="
        width: 300px;
        padding: 5px;
        border: 6px solid rgba(255, 255, 255, 0.781);
        box-shadow: 30px 30px 30px rgba(205, 201, 191, 0.854);
      "
      type="text"
      v-model="searchText"
      @input="filterCountries"
      placeholder="Search for a country..."
    />
    <select
      v-model="selectedContinent"
      style="
        margin-left: 650px;
        padding: 5px;
        border: 7px solid rgba(255, 255, 255, 0.824);
        box-shadow: 30px 30px 30px rgba(229, 223, 204, 0.854);
      "
    >
      <option value="">Filterr by Region</option>
      <option
        v-for="(continent, index) in continents"
        :value="continent"
        :key="index"
      >
        {{ continent }}
      </option>
    </select>
  </div>
  <div
    style="
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      row-gap: 20px;
    "
    v-if="!selectedCountry"
  >
    <div v-for="country in filteredCountries" :key="country.name">
      <img
        style="width: 350px; height: 200px"
        :src="country.flags.png"
        :alt="country.name"
        @click="showDetails(country)"
      />
      <div
        style="
          border: 1px solid rgba(255, 255, 255, 0.881);
          box-shadow: 30px 30px 30px rgba(229, 223, 204, 0.854);
        "
      >
        <h2 style="color: black">{{ country.name.common }}</h2>
        <br />
        <div>
          <span style="color: black; font-size: 20px">Population:</span
          ><span>{{ country.population }}</span>
        </div>
        <div>
          <span style="color: black; font-size: 20px">Capital:</span
          ><span>{{ country.capital }}</span>
        </div>
        <div>
          <span style="color: black; font-size: 20px">Region: </span
          ><span>{{ country.region }}</span>
        </div>
      </div>
    </div>
  </div>
  <button v-if="showDetailsPage" @click="goBack">Back</button>
  <br />
  <br />
  <div v-if="selectedCountry">
    <div style="display: flex; gap: 100px">
      <img
        :src="selectedCountry.flags.png"
        :alt="selectedCountry.name"
        style="width: 350px; height: 250px"
      />
      <div>
        <h2>{{ selectedCountry.name.common }}</h2>
        <br />
        <div style="display: flex; justify-content: space-between">
          <p>Population: {{ selectedCountry.population }}</p>
          <p>Capital: {{ selectedCountry.capital }}</p>
        </div>
        <br />
        <div style="display: flex; justify-content: space-between">
          <p>Region: {{ selectedCountry.region }}</p>
          <p>Sub Region: {{ selectedCountry.subregion }}</p>
        </div>
        <br />
        <div style="display: flex; justify-content: space-between">
          <p>Native Name: {{ selectedCountry.name.official }}</p>
          <p>Top Level Domain: {{ selectedCountry.tld }}</p>
        </div>
        <br />
        <div style="display: flex; gap: 300px">
          <p>Currencies: {{ selectedCountry.currencies }}</p>
          <p style="max-width: 300px">
            Languages: {{ selectedCountry.languages }}
          </p>
        </div>
        <br />
        <div style="display: flex">
          <p>Borders:</p>
          <div
            v-for="(border, index) in selectedCountry.borders"
            :key="'border' + index"
          >
            <button @click="showBorderDetails(border)">
              {{ border }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
.dark-mode {
  background-color: rgba(0, 0, 0, 0.934);
  color: beige;
}
.flex {
  display: flex;
  margin-bottom: 100px;
  justify-content: space-between;
  border-bottom: 1px solid rgba(255, 255, 255, 0.824);
  box-shadow: 30px 30px 30px rgba(229, 223, 204, 0.854);
}
</style>
