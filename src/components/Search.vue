<template>
  <div class="container is-search-box is-list-search">
    <div class="columns is-mobile">
      <div class="column is-search-header">
        <img alt="logo" src="~@/assets/logo-search.png" class="logo">
      </div>
    </div>
    <div class="columns is-mobile list-search is-multiline">
      <div class="column is-full">
        <h1 class="is-size-3 m-b-10">Global Situation</h1>
        <div class="global-date">
          <span>Data Update: <span class="has-text-weight-normal">{{latestDateUpdate}}</span></span>
        </div>
      </div>
      <div class="column" v-for="(latest, i) in latestData" :key="`covid-${i}`" :class="latest.class">
        <div class="card">
          <div class="card-content">
            <p class="title is-capitalized">{{latest.name}}</p>
            <p class="subtitle">{{latest.total}}</p>
          </div>
        </div>
      </div>
      <div class="column is-full">
        <hr>
        <h1 class="is-size-3 m-b-10">Search by Country</h1>
        <b-field>
          <div class="control has-icons-right">
            <input v-model="search" class="input is-medium" type="search" placeholder="Search by Country..." v-on:keyup.enter="goToSearch()">
            <span class="is-input-button is-right">
              <b-button type="is-primary" @click="goToSearch()">Search</b-button>
            </span>
          </div>
        </b-field>
      </div>
      <div class="column is-full" v-if="loading">
        <b-loading :is-full-page="true" :active.sync="loading" :can-cancel="false"></b-loading>
      </div>
      <div class="column is-full" v-else-if="!loading">
        <b-table :data="listData">
          <template v-if="!loading && !listData.length" #empty>
            <div class="empty-data">
              <img src="~@/assets/no-data-found.png" alt="no data found" width="140" class="m-b-20">
              <p class="is-size-4">No data found</p>
            </div>
          </template>

          <template #default="props">
            <b-table-column field="country_name" label="Country" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              <span>{{ props.row.country_name }}</span>
            </b-table-column>
            <b-table-column field="active_cases" label="Active Cases" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.active_cases }}
            </b-table-column>
            <b-table-column field="cases" label="Cases" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.cases }}
            </b-table-column>
            <b-table-column field="new_cases" label="New_Cases" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.new_cases }}
            </b-table-column>
            <b-table-column field="deaths" label="Deaths" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.deaths }}
            </b-table-column>
            <b-table-column field="new_deaths" label="New Deaths" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.new_deaths }}
            </b-table-column>
            <b-table-column field="serious_critical" label="Serious Critical" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.serious_critical }}
            </b-table-column>
            <b-table-column field="total_tests" label="Total Tests" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.total_tests }}
            </b-table-column>
            <b-table-column field="total_recovered" label="Total Recovered" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.total_recovered }}
            </b-table-column>
          </template>
        </b-table>
      </div>
      <div class="column" v-else>kosong</div>
    </div>
    <p>@MuhammadKurnia</p>
  </div>
</template>

<script>
import moment from 'moment'

const COVID_API = 'https://coronavirus-monitor.p.rapidapi.com/';
const RAPIDAPI_HOST = 'coronavirus-monitor.p.rapidapi.com'
const RAPIDAPI_KEY = 'de7fba554emshfb9b74c9826879ep1cb6bcjsn41ac9441cbc4'

export default {
  name: 'Search',
  data() {
    const today = new Date()
    return {
      search: '',
      loading: false,
      globalLatestData: {},
      latestData: {},
      listData: [],
      latestDateUpdate: '',
      date: new Date(),
      maxDate: new Date(today.getFullYear(), today.getMonth(), today.getDate())
    }
  },
  mounted() {
    this.getLatestTotal();
    this.getSearch('/cases_by_country.php', '');
  },
  methods: {
    constructDate(val) {
      const date = moment(val).format("DD-MM-YYYY")
      return this.latestDateUpdate = date
    },
    getLatestTotal() {
      return fetch(`${COVID_API}coronavirus/worldstat.php`, {
        method: "GET",
        "headers": {
          "x-rapidapi-host": RAPIDAPI_HOST,
          "x-rapidapi-key": RAPIDAPI_KEY,
        }
      })
      .then(response => response.json())
      .then(response => {
        this.constructDate(response.statistic_taken_at);
        this.latestData = [
          {class: 'is-one-quarter active-case', name: 'Active Case', total: response.active_cases},
          {class: 'is-one-quarter new-case', name: 'New Case', total: response.new_cases},
          {class: 'is-one-quarter new-death', name: 'New Deaths', total: response.new_deaths},
          {class: 'is-one-quarter serious-critical', name: 'Serious Critical', total: response.serious_critical},
          {class: 'total-cases', name: 'Total Cases', total: response.total_cases},
          {class: 'total-death', name: 'Total Deaths', total: response.total_deaths},
          {class: 'total-recovered', name: 'Total Recovered', total: response.total_recovered}
        ];
      })
      .catch(err => {
        this.$buefy.notification.open({
          duration: 5000,
          message: `please check your internet connection and try again`,
          position: 'is-bottom-right',
          type: 'is-danger',
          hasIcon: false
        })
        throw err
      })
    },
    getSearch(endpoint, query) {
      this.loading = true;
      return fetch(`${COVID_API}/coronavirus${endpoint}${query}`, {
        method: "GET",
        "headers": {
          "x-rapidapi-host": RAPIDAPI_HOST,
          "x-rapidapi-key": RAPIDAPI_KEY,
        }
      })
      .then(response => response.json())
      .then(response => {
        if(!query) {
          this.listData = response.countries_stat;
        } else {
          this.listData = response.latest_stat_by_country;
        }
      })
      .catch(err => {
        this.$buefy.notification.open({
          duration: 5000,
          message: 'please check your internet connection and try again',
          position: 'is-bottom-right',
          type: 'is-danger',
          hasIcon: false
        })
        throw err
      })
      .finally(() => {
        this.loading = false
      })
    },
    goToSearch() {
      const searchValue = this.search;
      const endpoint = '/latest_stat_by_country.php?country='

      this.getSearch(endpoint, searchValue);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
