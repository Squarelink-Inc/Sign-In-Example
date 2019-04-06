<template>
  <div class="hello">
    <h4 class="title">Display all of your available account info using the implicit grant<br> <i>(response_type=token)</i></h4>
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
      url: `${config.APP_URL}/authorize?`+
      	`client_id=${config.CLIENT_ID}`+
      	`&scope=[user,user:name,user:email,user:security,wallets:edit,wallets:create,wallets:remove,wallets:read]`+
      	`&redirect_uri=${config.REDIRECT}`+
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


    getWallet(access_token) {
      return axios.get(`${config.API_URL}/wallet`, {
        params: {
          access_token,
          wallet_id: ''
        }
      }).then(({ data }) => {
        //console.log(data)
      })
    },
    editWallet(access_token) {
      return axios.patch(`${config.API_URL}/wallet`, {
        access_token,
        wallet_id: '',
        name: ''
      }).then(({ data }) => {
        console.log(data)
      })
    },
    createWallet(access_token) {
      return axios.post(`${config.API_URL}/wallet`, {
        access_token,
        name: ''
      }).then(({ data }) => {
        console.log(data)
      })
    },
    removeWallet(access_token) {
      return axios.delete(`${config.API_URL}/wallet`, {
        params: {
          access_token,
          wallet_id: ''
        }
      }).then(({ data }) => {
        console.log(data)
      }).catch(err => {
        console.log(err.response.data)
      })
    },
    getName(access_token) {
      return axios.get(`${config.API_URL}/user/name`, {
        params: { access_token }
      }).then(({ data }) => {
        console.log(data)
      })
    },
    getEmail(access_token) {
      return axios.get(`${config.API_URL}/user/email`, {
        params: { access_token }
      }).then(({ data }) => {
        console.log(data)
      })
    },
    getSecurity(access_token) {
      return axios.get(`${config.API_URL}/user/security`, {
        params: { access_token }
      }).then(({ data }) => {
        console.log(data)
      })
    },




    getWallets(access_token) {
      return axios.get(`${config.API_URL}/wallets`, {
        params: {
          access_token
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
      return axios.get(`${config.API_URL}/txs`, {
        params: {
          access_token,
        }
      }).then(({ data }) => {
        const { txs } = data
        this.txs = txs
      }).catch(err => {
        const { message } = err.response.data
        this.error = message
      })
    },
    getTx(access_token) {
      return axios.get(`${config.API_URL}/tx`, {
        params: {
          access_token,
          id: '0x05ae19a5199caa5256c36fa9c58aa4db4e183c9861d9b277ef2be2d045cc329',
          network: 'custom',
          network_config: {
            url: 'http://localhost:8545'
          }
        }
      }).then(({ data }) => {
        console.log(data.tx)
      }).catch(err => {
        const { message } = err.response.data
        console.log(message)
      })
    },
    getUser(access_token) {
      return axios.get(`${config.API_URL}/user`, {
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
      this.getTxs(access_token)
      this.getUser(access_token)
      this.getWallets(access_token)

      // Testing
      //this.getWallet(access_token)
      //this.editWallet(access_token)
      //this.createWallet(access_token)
      //this.removeWallet(access_token)
      //this.getName(access_token)
      //this.getEmail(access_token)
      //this.getSecurity(access_token)
      //this.getTx(access_token)
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
