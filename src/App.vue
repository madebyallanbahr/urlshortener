<template>
  <form class="uk-container uk-position-center" @submit.prevent="generateURL">
    <h2 class="uk-text-large uk-text-bold">URL Shortener</h2>
    <p class="uk-text-muted">Encurtador de links personalizados com a nossa marca!</p>
    <input v-model="url" type="text" placeholder="URL" class="uk-input uk-margin" />
    <button type="submit" class="uk-button uk-button-primary">Encurtar</button>
  </form>

  <div class="uk-container uk-margin uk-text-center uk-position-bottom-center">
    <table
      v-if="shortURL.length != 0 && fullURL.length != 0"
      class="uk-table-justify uk-margin uk-table uk-table-hover uk-table-striped"
    >
      <thead>
        <tr>
          <th>URL Original</th>
          <th>URL Encurtada</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(url, index) in fullURL" :key="index">
          <!-- Coluna para URL Original -->
          <td>{{ url }}</td>
          <!-- Coluna para URL Encurtada -->
          <td>{{ shortURL[index] }}</td>

          <!-- Coluna com ícone de link para redirecionamento -->
          <td @click="copyToClipboard(shortURL[index])">
            <button><span class="uk-icon" uk-icon="link"></span></button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>

  <div class="uk-margin uk-position-top-center uk-text-center uk-padding-small">
    <stripe-buy-button :buy-button-id="buttonPayAPIKey" :publishable-key="paymentAPIKey">
    </stripe-buy-button>
  </div>

  <div
    v-if="error && shortURL.length == 0"
    class="uk-alert-danger uk-margin uk-position-bottom-left uk-text-center"
    uk-alert
  >
    <a href="#" class="uk-alert-close"></a>
    <p class="uk-text-danger">{{ error }}</p>
  </div>
  <div
    v-if="success && shortURL.length > 0"
    class="uk-alert uk-margin uk-position-bottom-left uk-text-center"
    uk-alert
  >
    <a href="#" class="uk-alert-close"></a>
    <p class="uk-margin">{{ success }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const paymentAPIKey = ref(import.meta.env.VITE_PAYMENT_ID)
const buttonPayAPIKey = ref(import.meta.env.VITE_BUTTON_PAY_ID)

const success = ref('')
const error = ref('')
const url = ref('')
const shortURL = ref([])
const fullURL = ref([])

const generateURL = async () => {
  if (!url.value) {
    error.value = 'O url não foi informado!'
    setTimeout(() => {
      error.value = ''
    }, 3000)
    return
  }
  try {
    const response = await fetch('https://www.urlshortener.com.br/api/short', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ url: url.value })
    })

    const data = await response.json()

    if (!response.ok) {
      error.value = data.error
      setTimeout(() => {
        error.value = ''
      }, 3000)
      return
    }

    fullURL.value.push(url.value)
    shortURL.value.push(data.short)
  } catch (error) {
    console.error('Error:', error.message)
  }
}
const copyToClipboard = (fUrl) => {
  navigator.clipboard
    .writeText(`https://www.urlshortener.com.br/${fUrl}`)
    .then(() => {
      success.value = 'Link copiado para área de transferência.'
    })
    .catch((err) => {
      error.value = `Não foi possivel copiar o link devido a: ${err} `
    })

  setTimeout(() => {
    success.value = ''
    error.value = ''
  }, 3000)
}
</script>
