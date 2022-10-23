<script setup>
import axios from "axios";
import { ref, watch } from "vue";

const searchedAddress = ref("");
const data = ref([]);
const dpe = ref([]);
const dpe_new = ref([]);
const dpe_tertiaire = ref([]);


function callBanAPI(event) {
  let address = event.target.value;

  axios
    .get(`https://api-adresse.data.gouv.fr/search/?q=${address}&limit=7`)
    .then((res) => (data.value = res.data.features));
}

function callDpeAPI(id_ban) {
  let id = id_ban;

  axios
    .get(
      `https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-existants/lines?qs=(Identifiant__BAN:"${id}")`
    )
    .then((res) => (dpe.value = res.data.results));

  axios
    .get(
      `https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-neufs/lines?qs=(Identifiant__BAN:"${id}")`
    )
    .then((res) => (dpe_new.value = res.data.results));

  axios
    .get(
      `https://data.ademe.fr/data-fair/api/v1/datasets/dpe-v2-logements-neufs/lines?qs=(Identifiant__BAN:"${id}")`
    )
    .then((res) => (dpe_tertiaire.value = res.data.results));

}
</script>

<template>
  <form>
    <div class="relative w-full">
      <input type="search" id="default-search" @input="callBanAPI($event)" class="
          block
          p-4
          pl-10
          w-full
          text-sm text-gray-900
          bg-gray-50
          rounded-lg
          border border-gray-300
          focus:ring-blue-500 focus:border-blue-500
          dark:bg-gray-700
          dark:border-gray-600
          dark:placeholder-gray-400
          dark:text-white
          dark:focus:ring-blue-500
          dark:focus:border-blue-500
        " placeholder="20 avenue de Ségur, Paris" required />

      <li v-for="address in data" :key="address.properties.id" @click="callDpeAPI(address.properties.id)">
        {{ address.properties.label }} - {{ address.properties.id }}
      </li>

      <div class="flex ...">
        <div class="w-1/3">
          <h2>dpe-v2-logements-existants</h2>
          <p>{{ dpe }}</p>
        </div>
        <div class="w-1/3">
          <h2>dpe-v2-logements-neufs</h2>
          <p>{{ dpe_new }}</p>
        </div>
        <div class="w-1/3">
          <h2>dpe_tertiaire</h2>
          <p>{{ dpe_tertiaire}}</p>
        </div>
      </div>
    </div>
  </form>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
