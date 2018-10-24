<template>
  <div class="hello">
    <h2 class="title">Display all of your available account info <i>(using implicit grant)</i></h2>
    <a v-bind:href="url">
      <img src="https://squarelink.com/img/sign-in.svg" width="240"/>
    </a>
    <div class="data">
      <div v-if="error" class="error">{{error}}</div>
      <div v-if="user" class="category">
        <h3 class="title">GET /user</h3>
        <vue-json-pretty :data="user"/>
      </div>
      <div v-if="wallets" class="category">
        <h3 class="title">GET /wallets</h3>
        <vue-json-pretty :data="wallets"/>
      </div>
      <div v-if="txs" class="category">
        <h3 class="title">GET /txs</h3>
        <vue-json-pretty :data="txs"/>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import axios from 'axios'
import VueJsonPretty from 'vue-json-pretty'
import config from '../config'

export default {
  name: 'HelloWorld',
  components: { VueJsonPretty },
  data() {
    return {
      url: `https://app.squarelink.com/authorize?`+
      	`client_id=${config.client_id}`+
      	`&scope=[user,wallets:read]`+
      	`&redirect_uri=${config.redirect}`+
      	`&state=abcdef`+
      	`&response_type=token`,
      wallets: null,
      txs: null,
      user: null,
      error: ''
    }
  },
  methods: {
    getUrlVars() {
      var vars = {}
      var parts = window.location.href.replace(/[?&]+([^=&]+)=([^&]*)/gi, function(m,key,value) {
        vars[key] = value;
      })
      return vars
    },
    getWallets(access_token) {
      return axios.get('https://api.squarelink.com/wallets', {
        params: {
          access_token,
          currencies: 'ETH;BTC;LTC',
          erc20: true
        }
      }).then(({ data }) => {
        const { success, ...wallets } = data
        this.wallets = wallets
      }).catch(err => {
        const { message } = err.response.data
        this.error = message
      })
    },
    getTxs(access_token) {
      return axios.get('https://api.squarelink.com/txs', {
        params: {
          access_token,
          currencies: 'ETH;BTC;LTC',
          all_erc20: true
        }
      }).then(({ data }) => {
        const { txs } = data
        this.txs = txs
      }).catch(err => {
        const { message } = err.response.data
        this.error = message
      })
    },
    getUser(access_token) {
      return axios.get('https://api.squarelink.com/user', {
        params: { access_token }
      }).then(({ data }) => {
        const { success, ...info } = data
        this.user = info
      }).catch(err => {
        const { message } = err.response.data
        this.error = message
      })
    },
  },
  mounted() {
    const { access_token, error } = this.getUrlVars()
    if (access_token) {
      this.getWallets(access_token)
      this.getTxs(access_token)
      this.getUser(access_token)
    } else if (error) {
      this.error = error
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.data {
  width: 500px;
  margin: 0 auto;
}

.error {
  color: red;
}

.title {
  text-align: center;
}
.category{
  text-align: left;
}

</style>
