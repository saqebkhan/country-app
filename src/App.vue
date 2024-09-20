<template>
  <div id="app" class="container">
    <h1 class="title">Country Manager</h1>
    <select
      v-model="selectedCountryName"
      @change="fetchCountryDetails"
      v-if="countries.length"
      class="country-select"
    >
      <option
        v-for="country in countries"
        :key="country.name"
        :value="country.name"
      >
        {{ country.name }}
      </option>
    </select>

    <div v-if="countryDetails" class="country-details">
      <h2 class="country-name">{{ countryDetails.name }}</h2>
      <img
        :src="`http://localhost:8080/${countryDetails.flag}`"
        v-if="countryDetails.flag"
        class="country-flag"
      />
      <p class="country-rank">Rank: {{ countryDetails.rank }}</p>
    </div>

    <h3 class="form-title">Add New Country</h3>
    <form @submit.prevent="addCountry" class="country-form">
      <input
        v-model="newCountry.name"
        placeholder="Country Name"
        required
        minlength="3"
        maxlength="20"
        class="input-field"
      />
      <input
        v-model="newCountry.rank"
        type="number"
        placeholder="Rank"
        required
        class="input-field"
      />
      <select v-model="newCountry.continent" required class="input-field">
        <option
          v-for="continent in uniqueContinents"
          :key="continent"
          :value="continent"
        >
          {{ continent }}
        </option>
      </select>
      <input
        type="file"
        @change="handleFileUpload"
        accept="image/jpeg,image/png"
        class="file-input"
      />
      <button type="submit" class="submit-button">Add Country</button>
    </form>
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      countries: [],
      selectedCountryName: "",
      countryDetails: null,
      newCountry: {
        name: "",
        rank: "",
        continent: "",
      },
      errorMessage: "",
      uniqueContinents: [
        "Asia",
        "Africa",
        "North America",
        "South America",
        "Europe",
        "Oceania",
      ],
    };
  },
  created() {
    this.loadCountries();
  },
  methods: {
    async loadCountries() {
      const response = await axios.get("http://localhost:8080/countries");
      this.countries = response.data;
      if (this.countries.length) {
        this.selectedCountryName = this.countries[0].name; // Set the first country's name
        await this.fetchCountryDetails();
      }
    },
    async fetchCountryDetails() {
      if (this.selectedCountryName) {
        const response = await axios.get(
          `http://localhost:8080/country/${this.selectedCountryName}`
        );
        this.countryDetails = response.data;
        console.log(this.countryDetails);
      }
    },
    async addCountry() {
      const formData = new FormData();
      formData.append("name", this.newCountry.name);
      formData.append("rank", this.newCountry.rank);
      formData.append("continent", this.newCountry.continent);
      formData.append("image", this.file);

      try {
        await axios.post("http://localhost:8080/country", formData);
        this.loadCountries();
        this.newCountry = { name: "", rank: "", continent: "" };
        this.errorMessage = "";
      } catch (error) {
        this.errorMessage = error.response.data.error || "An error occurred";
      }
    },
    handleFileUpload(event) {
      this.file = event.target.files[0];
    },
    async deleteLastFourCountries() {
      try {
          const response = await axios.delete("http://localhost:8080/countries/deleteLastFour");
          console.log(response.data.message);
          // Optionally reload the countries after deletion
          this.loadCountries();
      } catch (error) {
          console.error("An error occurred while deleting countries:", error.response.data.error);
      }
  }
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.title {
  text-align: center;
  color: #333;
}

.country-select, .input-field, .file-input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
}

.country-details {
  margin: 20px 0;
  text-align: center;
}

.country-name {
  font-size: 24px;
  font-weight: bold;
}

.country-flag {
  width: 150px;
  height: auto;
  margin: 10px 0;
}

.country-rank {
  font-size: 18px;
  color: #555;
}

.form-title {
  margin-top: 30px;
  font-size: 20px;
  color: #444;
}

.submit-button {
  background-color: #28a745;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.submit-button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  text-align: center;
}
</style>
