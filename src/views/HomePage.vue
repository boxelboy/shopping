<script setup>
import { onMounted, ref } from 'vue'
import { IonContent, IonHeader, IonPage, IonIcon, IonTitle, IonToolbar } from '@ionic/vue'
import {addIcons} from "ionicons"
import { trashOutline, checkmarkOutline, closeCircleOutline, refreshOutline } from 'ionicons/icons'

addIcons({ trashOutline, checkmarkOutline, closeCircleOutline, refreshOutline })

const userName = import.meta.env.VITE_USER
const pwd = import.meta.env.VITE_PWD
const memo = import.meta.env.VITE_URL
const signIn = import.meta.env.VITE_SIGN_IN

const listArr = ref([])
const newItem = ref('')
const renderKey = ref(0)

const addNewItem = async () => {
  listArr.value.push(newItem.value)
  newItem.value = ''
  await updateShopping(convertToString(listArr))
}

const convertToString = (value) => {
  let str = ''
  value.value.forEach((item, index) => {
    str += item
    str += (index !== value.value.length - 1) ? '\n' : ''
  })
  return str
}

const doRefresh = async () => {
  console.log('refresh')
  const shopping = await getShopping()
  makeList(shopping)
  renderKey.value += 1
};

const leaveApp = () => {
  navigator.app.exitApp();
}

const getToken = async () => {
  try {
    const response = await fetch(signIn, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        passwordCredentials: { username: userName, password: pwd }
      })
    })

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    const data = await response.json()
    return data.accessToken
  } catch (error) {
    console.error('Failed to fetch token:', error)
    return null
  }
}

const getShopping = async () => {
  const token = await getToken()
  if (!token) {
    console.error('Authentication failed: No token received.')
    return null
  }

  try {
    const response = await fetch(memo, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    })

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    return await response.json()
  } catch (error) {
    console.error('Failed to fetch memo:', error)
    return null
  }
}

const makeList = (shopping) => {
  if (shopping?.content) {
    listArr.value = shopping.content.split('\n')
  } else {
    listArr.value = []
  }
}

const removeFromList = async (idx) => {
  listArr.value.splice(idx, 1)
  await updateShopping(convertToString(listArr))
}

const updateShopping = async (updShopping) => {
  const token = await getToken()
  if (!token) {
    console.error('Authentication failed: No token received.')
    return null
  }

  try {
    const response = await fetch(memo, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        "state": "STATE_UNSPECIFIED",
        "content": updShopping,
        "visibility": "VISIBILITY_UNSPECIFIED"
      })
    })

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    const data = await response.json()
    return data.content?.split('\n') || []
  } catch (error) {
    console.error('Failed to update memo:', error)
    return null
  }
}

onMounted(async () => {
  const shopping = await getShopping()
  makeList(shopping)
})
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Shopping List</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="leaveApp()">
            <ion-icon name="close-circle-outline"></ion-icon>
          </ion-button>
        </ion-buttons>

      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Shopping List</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container" :key="renderKey">
        <p v-for="(item, idx) in listArr" :key="idx">
          {{ item || 'Loading...' }} <ion-icon name="trash-outline" @click="removeFromList(idx)"></ion-icon>
        </p>
        <p>
          <input placeholder="Enter new item" v-model="newItem" autocomplete="off" /> <ion-icon class="tick" name="checkmark-outline" @click="addNewItem()"></ion-icon>
        </p>
      </div>
    </ion-content>
    <div class="refresh"><ion-icon name="refresh-outline" @click="doRefresh()"></ion-icon></div>
  </ion-page>
</template>

<style scoped lang="scss">
.title-default {
  color: #8c8c8c;
}
.sc-ion-buttons-md-s .button-clear {
  margin-right: 10vw;
}
#container {
  margin-top: 3vh;
  text-align: left;
  position: relative;
  width: 75vw;
  padding: 0 0 0 2vw;
  font-family: Roboto, sans-serif;

  strong { font-size: 20px; line-height: 26px; }
  p {
    font-size: 16px;
    line-height: 22px;
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    margin: 0 0 1vh 0;

    &:last-child {
      margin-top: 2vh;
    }

    i {
      cursor: pointer;
    }
  }
  a { text-decoration: none; }

  button {
    padding: 0.5vh 2vw;
    border-radius: 1vw;
  }

  input {
    border: 1px solid #000;
    background: transparent;
    color: #000;
    width: 50vw;

    &::placeholder {
      color: #000;
      font-style: italic;
    }
  }
}

.tick {
  font-weight: 900;
  font-size: 20px;
  color: #333333;
}

.refresh {
  position: absolute;
  right: 20px;
  bottom: 40px;
  color: #000;
  font-size: 30px;
}
</style>