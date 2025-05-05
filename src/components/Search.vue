<style scoped></style>
<template>
  <div>
    <input v-model="name" />#
    <input v-model="tag" />
    <select v-model="region">
      <option value="NA">NA</option>
      <option value="EUNE">EUNE</option>
      <option value="EUW">EUW</option>
    </select>
    <button @click="getTop()">Search</button>
    <div>{{ topChamp }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

var name = "";
var tag = "";
var region = "";
const id = ref("");
const apiKey = "RGAPI-781d28ec-5973-4c35-9759-772433b48d51";
const champsJson = ref("");
const champMap = ref(new Map());
const topChamp = ref("");
onMounted(() => {
  loadChamps();
});

async function loadChamps() {
  var URL =
    "https://ddragon.leagueoflegends.com/cdn/15.9.1/data/en_US/champion.json";
  const response = await fetch(URL);
  const data = await response.json();
  champsJson.value = data.data;

  const map = new Map();
  for (const champName in champsJson.value) {
    const champion = champsJson.value[champName];
    map.set(champion.key, champion);
  }
  champMap.value = map;
}

async function getTop() {
  await getPlayer();
  const response = await fetch(
    `https://euw1.api.riotgames.com/lol/champion-mastery/v4/champion-masteries/by-puuid/${id.value}?api_key=${apiKey}`
  );
  const data = await response.json();

  topChamp.value = champMap.value.get(data[0].championId.toString()).id;
}

async function getPlayer() {
  const URL = `https://${
    region === "EUNE" || region === "EUW" ? "europe" : "americas"
  }.api.riotgames.com/riot/account/v1/accounts/by-riot-id/${name}/${tag}?api_key=${apiKey}`;

  console.log(URL);

  const response = await fetch(URL);
  const data = await response.json();
  id.value = data.puuid;
  console.log(id.value);
}
</script>
