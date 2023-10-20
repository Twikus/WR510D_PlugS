<script setup lang="ts">
import { onMounted, ref } from 'vue'

interface ShellyState {
  relays: { ison: boolean }[];
  temperature: number;
  wifi_sta: { ip: string };
  meters: { power: number }[];
}

const state = ref<ShellyState>({
  relays: [{ ison: false }],
  temperature: 0,
  wifi_sta: { ip: '' },
  meters: [{ power: 0 }]
});

const showOnButton = ref(false)
const showOffButton = ref(false)


onMounted(() => {
  getShellyStatus()
})

const getShellyStatus = async () => {
  const response = await fetch('http://192.168.1.100/status')
  state.value = await response.json()
  console.log(state.value)

  if (state.value.relays[0].ison === true) {
    showOffButton.value = true
    showOnButton.value = false
  } else {
    showOnButton.value = true
    showOffButton.value = false
  }
}

const setOnShellyPlug = async () => {
  // requete pour passer data.ison en true
  const response = await fetch('http://192.168.1.100/relay/0?turn=on')
  const data = await response.json()
  getShellyStatus()
}

const setOffShellyPlug = async () => {
  // requete pour passer data.ison en false
  const response = await fetch('http://192.168.1.100/relay/0?turn=off')
  const data = await response.json()
  getShellyStatus()
}

</script>

<template>
  <header>
    <h1>Administration prise Shelly PlugS</h1>
    <div v-if="state">
      <div class="container-info">
        <div>
          <h3>Status :</h3>
          <p v-if="state && showOffButton">On</p>
          <p v-if="state && showOnButton">Off</p>
        </div>
        <div>
          <div>
            <h3>Temperature :</h3>
            <p v-if="state">{{ state.temperature }}</p>
          </div>
          <div>
            <h3>IP Adress :</h3>
            <p v-if="state">{{ state.wifi_sta.ip }}</p>
          </div>
          <div>
            <h3>Power :</h3>
            <p v-if="state">{{ state.meters[0].power }}W</p>
          </div>
        </div>
      </div>

      <br><br><br>
      
      <button v-if="showOnButton" @click="setOnShellyPlug">Allumer</button>
      <button v-if="showOffButton" @click="setOffShellyPlug">Éteindre</button>
    </div>
    <div v-else>
      <h1>Chargement en cours...</h1>
    </div>
  </header>
</template>

<style scoped lang="scss">
  .container-info {
    display: flex;
    justify-content: space-between;

    h3 {
      color: hsla(160, 100%, 37%, 1);
    }

    div {
      display: flex;
      flex-direction: column;
      align-items: center;

      & > div {
        display: flex;
        flex-direction: row;
        align-items: center;
      }
    }
  }
</style>