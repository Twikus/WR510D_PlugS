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
  const response = await fetch('https://shelly-77-eu.shelly.cloud/device/status?auth_key=MWNiMjY5dWlk404459961993DCA83AE44BC6E3A6F58906952E7BECA0A5B69DC375C964915ACBC0EA536A0639CB73&id=4022d88e30e8')

  state.value = await response.json()
  state.value = state.value.data.device_status

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
  const formdata = new FormData();
  formdata.append("channel", "0");
  formdata.append("turn", "on");
  formdata.append("id", "4022d88e30e8");
  formdata.append("auth_key", "MWNiMjY5dWlk404459961993DCA83AE44BC6E3A6F58906952E7BECA0A5B69DC375C964915ACBC0EA536A0639CB73");

  const requestOptions = {
    method: 'POST',
    body: formdata,
  };

  await fetch("https://shelly-77-eu.shelly.cloud/device/relay/control", requestOptions)
    .then(response => response.json())
    .then(result => console.log(result))
    .catch(error => console.log('error', error));

  getShellyStatus();
}

const setOffShellyPlug = async () => {
  const formdata = new FormData();
  formdata.append("channel", "0");
  formdata.append("turn", "on");
  formdata.append("id", "4022d88e30e8");
  formdata.append("auth_key", "MWNiMjY5dWlk404459961993DCA83AE44BC6E3A6F58906952E7BECA0A5B69DC375C964915ACBC0EA536A0639CB73");

  const requestOptions = {
    method: 'POST',
    body: formdata,
  };

  fetch("https://shelly-77-eu.shelly.cloud/device/relay/control", requestOptions)
    .then(response => response.text())
    .then(result => console.log(result))
    .catch(error => console.log('error', error));

  getShellyStatus();
}

</script>

<template>
  <div>
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
  </div>
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