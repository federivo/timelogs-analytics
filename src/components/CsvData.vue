<template>

  <div class="px-4 sm:px-6 lg:px-8">
    <div class="sm:flex sm:items-center">
      <div class="sm:flex-auto">
        <h1 class="text-xl font-semibold text-gray-900">Timelogs analytics</h1>
        <p class="mt-2 text-sm text-gray-700">Timelogs... you know...</p>
      </div>
      <div class="flex space-x-1 rounded-lg bg-slate-100 p-0.5" role="tablist" aria-orientation="horizontal">
        <button class="flex items-center rounded-md py-[0.4375rem] pl-2 pr-2 text-sm font-semibold lg:pr-3" role="tab" type="button" tabindex="0" :class="{'bg-white shadow': detailIsActive}" @click="detailIsActive = !detailIsActive">
          <svg class="h-5 w-5 flex-none stroke-sky-500" fill="none" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" xmlns="http://www.w3.org/2000/svg">
            <path d="M17.25 10c0 1-1.75 6.25-7.25 6.25S2.75 11 2.75 10 4.5 3.75 10 3.75 17.25 9 17.25 10Z"></path>
            <circle cx="10" cy="10" r="2.25"></circle>
          </svg>
          <span class="sr-only lg:not-sr-only lg:ml-2 text-slate-900">Detail</span>
        </button>
        <button class="flex items-center rounded-md py-[0.4375rem] pl-2 pr-2 text-sm font-semibold lg:pr-3" role="tab" type="button" tabindex="-1" :class="{'bg-white shadow': !detailIsActive}" @click="detailIsActive = !detailIsActive">
          <svg class="h-5 w-5 flex-none stroke-slate-600" fill="none" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" xmlns="http://www.w3.org/2000/svg">
            <path d="m13.75 6.75 3.5 3.25-3.5 3.25M6.25 13.25 2.75 10l3.5-3.25"></path>
          </svg>
          <span class="sr-only lg:not-sr-only lg:ml-2 text-slate-600">Totals</span>
        </button>
      </div>
      <div class="mt-4 sm:mt-0 sm:ml-16 sm:flex-none">
        <input type="file" accept=".csv" @change="handleFileUpload( $event )" class="block w-full text-sm text-slate-500
          file:mr-4 file:py-2 file:px-4
          file:rounded-full file:border-0
          file:text-sm file:font-semibold
          file:bg-indigo-50 file:text-indigo-700
          hover:file:bg-indigo-100
        "/>
      </div>
    </div>

    <div class="mt-8 flex flex-col" :class="{'hidden': !detailIsActive}">
      <div class="-my-2 -mx-4 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="inline-block min-w-full py-2 align-middle md:px-6 lg:px-8">
          <div class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded-lg">
            <table v-if="parsed" class="divide-y divide-gray-300">
              <thead class="bg-gray-50">
                <tr class="bg-slate-100">
                  <th scope="col" class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6">Date</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900">User</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Project</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Issue</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900 w-400">Comment</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Hours</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200 bg-white">
              <tr v-for="(row, rowKey) in content.data"
                  v-bind:key="'row-'+rowKey">
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.Date }}</td>
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.User }}</td>
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.Project }}</td>
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.Issue }}</td>
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.Comment }}</td>
                <td class="px-2 py-2 text-sm text-gray-900">{{ row.Hours }}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <div class="mt-8 flex flex-col" :class="{'hidden': detailIsActive}">
      <div class="-my-2 -mx-4 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="inline-block  py-2 align-middle md:px-6 lg:px-8">
          <div class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded-lg">
            <table class="divide-y divide-gray-300">
              <thead class="bg-gray-50">
                <tr class="bg-slate-100">
                  <th scope="col" class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6">Date</th>
                  <th scope="col" class="px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Hours</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200 bg-white">
                <tr v-for="[key, total] in totals" :key="key">
                  <td class="px-2 py-2 text-sm text-gray-900">{{ key }}</td>
                  <td class="px-2 py-2 text-sm text-gray-900 text-right">{{ total }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import Papa from 'papaparse';
// import axios from 'axios';
// import initialJson from '../data/initial-data.json';

export default {

  name: "CsvData",

  data(){
    return {
      file: '',
      content: [],
      parsed: false,
      totals: [],
      detailIsActive: true,
      totalIsActive: false
    }
  },

  created(){
    this.content = JSON.parse('{"data":[]}');
    this.parsed = true;
    this.totals = this.calculateTotals(this.content.data);
    // console.log(this.totals);
  },

  methods: {
    parseFile(){
      Papa.parse( this.file, {
        header: true,
        skipEmptyLines: true,
        complete: function( results ){
          this.content = results;
          this.parsed = true;
          this.totals = this.calculateTotals(results.data);
        }.bind(this)
      } );
    },

    handleFileUpload( event ){
      this.file = event.target.files[0];
      this.parseFile();
    },

    calculateTotals(data) {
      let totals = new Map();
      data.forEach(item => {
        if (totals.get(item.Date) === undefined) {
          totals.set(item.Date, 0);
        }
        totals.set(item.Date, totals.get(item.Date) + parseFloat(item.Hours));
      })
      return totals;
    }
  }
}

</script>

<style scoped>

</style>