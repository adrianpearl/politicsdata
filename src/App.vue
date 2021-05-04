<template>
  <div class="grid grid-cols-1 md:grid-cols-4 lg:grid-cols-5 h-screen">
    <div class="bg-gray-100">
      <div class="m-6 flex flex-wrap justify-between md:flex-nowrap md:flex-col md:space-y-4 text-sm text-gray-600">
        <div class="flex flex-col">
          <label for="date_from">From Date</label>
          <input type="date" class="rounded-md border-0 md:w-full bg-gray-200" name="date_from" id="date_from" v-model="date_from"/>
        </div>
        <div class="flex flex-col">
          <label for="date_to">To Date</label>
          <input type="date" class="rounded-md border-0 md:w-full bg-gray-200" name="date_to" id="date_to" v-model="date_to"/>
        </div>
        <div class="flex flex-col">
          <button type="button" class="inline-flex md:w-full items-center justify-center p-2 rounded-md bg-gray-800 text-gray-400 hover:text-white hover:bg-gray-700" @click="csv">
            <div class="flex items-center space-x-2 mx-4">
              <svg class="min-h-5 w-5 inline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
              </svg>
              <span class="min-h-5 text-sm">Download CSV</span>
            </div>
          </button>
        </div>
      </div>
    </div>
    <div class="md:col-span-3 lg:col-span-4">
      <div class="m-4 overflow-x-auto">
        <table v-if="this.transactions.length > 0" class="table-fixed min-w-full">
          <thead>
            <tr class="bg-gray-200 text-gray-600 uppercase text-sm font-bold">
              <th
                class="py-3 px-6"
                v-for="(field, index) in Object.keys(this.transactions[0])"
                :key="index"
              >
                <div 
                  class="relative flex items-center space-x-1"
                >
                  <span>{{field}}</span>
                  <svg class="h-3 w-3 inline cursor-pointer" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" @click="panel = panel == field ? '' : field">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z" />
                  </svg>
                  <div 
                    class="origin-bottom-left absolute right-0 top-10 min-w-48 flex rounded-md shadow-lg p-2 bg-gray-100 ring-1 ring-black ring-opacity-5 transform z-10 text-right" 
                    :class="`ease-${panel==field ? 'out' : 'in'} duration-${panel==field ? '100' : '75'} opacity-${panel==field ? '100' : '0'} scale-${panel==field ? '100' : '95'} ${panel==field ? 'pointer-events-auto' : 'pointer-events-none'}`" 
                    role="menu" 
                    aria-orientation="vertical" 
                    aria-labelledby="options-menu"
                  >
                    <div class="relative mx-auto text-gray-600">
                      <input 
                        class="border-0 bg-gray-200 h-8 px-3 rounded-lg text-sm focus:outline-none focus:ring-1 text-gray-600"
                        type="search" 
                        name="search" 
                        placeholder="Search"
                      >
                    </div>
                    <button class="text-sm px-2 ml-2 text-white bg-green-400 hover:bg-green-500 rounded-md py-1">Search</button>
                  </div>
                </div>
              </th>
            </tr>
          </thead>
          <tbody v-if="!this.loading" class="text-gray-600 text-sm font-light">
            <tr
              class="border-b border-gray-200 hover:bg-gray-100"
              v-for="(transaction, row) in this.transactions"
              :key="(row+1)*Object.keys(transaction).length"
            >
              <td
                class="py-3 px-6 text-left"
                v-for="(value, col) in Object.keys(transaction).map(d => transaction[d])"
                :key="(row+1)*Object.keys(transaction).length + col + 1"
              >
                {{value}}
              </td>
            </tr>
          </tbody>
        </table>
        <div v-if="this.loading" class="flex justify-center w-full my-6">
          <div class="la-ball-clip-rotate-pulse la-dark">
            <div></div>
            <div></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
const fieldsjson = require('../fields.json');

export default {
  name: 'App',
  data() {
    return {
      transactions: [],
      errors: [],
      date_from: '',
      date_to: '',
      loading: false,
      fields: fieldsjson,
      panel: ''
    }
  },
  computed: {
    params() {
      let p = `select=${this.fields.filter(d => d.select).map(d => d.name).join(',')}`;
      // if (this.date_from) {
      //   p += `&date=gte.'${this.date_from}'`;
      // }
      // if (this.date_to) {
      //   p += `&date=lte.'${this.date_to}'`;
      // }
      return p;
    }
  },
  watch: {
    params() {
      this.fetchTransactions();
    }
  },
  methods: {
    async fetchTransactions() {
      this.loading = true;
      try {
        const response = await axios.get(`http://142.93.13.74:4000/transactions?${this.params}`);
        this.transactions = response.data;
      } catch (e) {
        this.errors.push(e);
      }
      this.loading = false;
    },
    csv() {
      if (this.transactions.length === 0) { return ''; }
      const keys = Object.keys(this.transactions[0]).map(d => d.includes(',') ? `"${d}"` : d);
      let str = "data:text/csv;charset=utf-8," + keys.join(',');
      for (const t of this.transactions) {
        str += '\r\n';
        str += keys.map(d => t[d].includes(',') ? `"${t[d]}"` : t[d]);
      }
      const encodedUri = encodeURI(str);
      window.open(encodedUri);
    }
  },
  async created() {
    this.fetchTransactions();
  }
}
</script>