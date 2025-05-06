<style scoped>
.splashHolder {
  display: flex;
  text-align: center;
  align-items: center;
  justify-content: center;
  flex-direction: row;
  gap: 2%;
}

.splash {
  display: flex;
  text-align: center;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  font-size: x-large;
}

.splash img {
  width: 100%;
}
</style>
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
    <div class="splashHolder" v-if="showImg">
      <div class="splash" v-for="champ in champions">
        <span>{{ champ }}</span>
        <img :src="getSplash(champ)">
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
const showImg = ref(false);
var name = "";
var tag = "";
var region = "";
const id = ref("");
const apiKey = "RGAPI-781d28ec-5973-4c35-9759-772433b48d51";
const champsJson = ref("");
const champMap = ref(new Map());
const champions = ref([])
const imgURL = ref("");
onMounted(() => {
  loadChamps();
});

function getSplash(champ) {
  console.log(champions);
  console.log(champ);
  return `https://ddragon.leagueoflegends.com/cdn/img/champion/loading/${champ}_0.jpg`;
}


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
  champions.value.length = 0;
  showImg.value = false;
  const response = await fetch(
    `https://${region.toLowerCase()}1.api.riotgames.com/lol/champion-mastery/v4/champion-masteries/by-puuid/${id.value}?api_key=${apiKey}`
  );
  const data = await response.json();

  for (var i = 0; i < 5; i++) {
    champions.value.push(champMap.value.get(data[i].championId.toString()).id);

  }
  showImg.value = true;
}

async function getPlayer() {
  const URL = `https://${region === "EUNE" || region === "EUW" ? "europe" : "americas"
    }.api.riotgames.com/riot/account/v1/accounts/by-riot-id/${name}/${tag}?api_key=${apiKey}`;


  const response = await fetch(URL);
  const data = await response.json();
  id.value = data.puuid;
  console.log(id.value);
}
</script>
