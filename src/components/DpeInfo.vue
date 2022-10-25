<script setup>
import axios from "axios";
import { ref, watch } from "vue";

const searchedAddress = ref("");
const data = ref([]);
const dpe = ref([]);


function callBanAPI(event) {
  let address = event.target.value;

  axios
    .get(`https://api-adresse.data.gouv.fr/search/?q=${address}&limit=10`)
    .then((res) => (data.value = res.data.features))
    .catch(function (error) {
      data.value = [];
    });
}

async function callDpeAPI(address_ban) {
  data.value = [];
  searchedAddress.value = address_ban.label;

  let id = address_ban.id;

  const axiosrequest1 = axios.get(`https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-existants/lines?qs=(Identifiant__BAN:"${id}")`);
  const axiosrequest2 = axios.get(`https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-neufs/lines?qs=(Identifiant__BAN:"${id}")`);
  const axiosrequest3 = axios.get(`https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-neufs/lines?qs=(Identifiant__BAN:"${id}")`);
  // you could also use destructuring to have an array of responses
  await axios.all([axiosrequest1, axiosrequest2, axiosrequest3]).then(axios.spread(function (res1, res2, res3) {
    const all_dpe = [...res1.data.results, ...res2.data.results, ...res3.data.results]

    const uniqueIds = [];

    const unique_dpe = all_dpe.filter(element => {
      const isDuplicate = uniqueIds.includes(element._id);

      if (!isDuplicate) {
        uniqueIds.push(element._id);

        return true;
      }
      return false;
    });

    dpe.value = unique_dpe
  }));
}
</script>

<template>
  <section class="relative w-[50rem] px-5 py-4 mx-auto rounded-md">
    <div class="relative">
      <span class="absolute inset-y-0 left-0 flex items-center pl-3">
        <svg class="w-5 h-5 text-gray-400" viewBox="0 0 24 24" fill="none">
          <path
            d="M21 21L15 15M17 10C17 13.866 13.866 17 10 17C6.13401 17 3 13.866 3 10C3 6.13401 6.13401 3 10 3C13.866 3 17 6.13401 17 10Z"
            stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
        </svg>
      </span>

      <input type="text" v-model=searchedAddress @input="callBanAPI($event)" @change="close_results"
        class="w-full py-3 pl-10 pr-4 text-gray-700 bg-white border rounded-md dark:bg-gray-900 dark:text-gray-300 dark:border-gray-600 focus:border-blue-400 dark:focus:border-blue-300 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-40"
        placeholder="20 avenue de Ségur, Paris">
    </div>

    <div v-if="data.length"
      class="absolute inset-x-0 px-6 py-3 mx-5  overflow-y-auto bg-white border rounded-md dark:bg-gray-900 dark:border-gray-700">
      <a v-for="address in data" :key="address.properties.id" @click="callDpeAPI(address.properties)"
        class="block py-1">
        <p class="mt-2 text-sm text-gray-500 dark:text-gray-400">{{ address.properties.label }} -
          {{address.properties.id }}</p>
      </a>
    </div>
  </section>

  <div v-for="el in dpe" :key="el._id" class="py-1">
    <div class="px-8 py-4 bg-white rounded-lg shadow-md dark:bg-gray-800">
      <div class="flex items-center justify-between">
        <span class="text-sm font-light text-gray-600 dark:text-gray-400">N°DPE : {{el["N°DPE"]}}</span>
      </div>
      <div class="flex items-center justify-between">
        <span class="text-sm font-light text-gray-600 dark:text-gray-400">Date_établissement_DPE :
          {{el.Date_établissement_DPE}}</span>
      </div>
      <div class="flex items-center justify-between">
        <span class="text-sm font-light text-gray-600 dark:text-gray-400">Etiquette_DPE : {{el.Etiquette_DPE}}</span>
      </div>
      <div class="flex items-center justify-between">
        <span class="text-sm font-light text-gray-600 dark:text-gray-400">Etiquette_GES : {{el.Etiquette_GES}}</span>
      </div>
      <div class="flex items-center justify-between">
        <span class="text-sm font-light text-gray-600 dark:text-gray-400">Adresse_(BAN) : {{el["Adresse_(BAN)"]}}</span>
      </div>
    </div>
  </div>

</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
