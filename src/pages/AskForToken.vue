<template>
  <v-container fluid fill-height>
    <v-layout align-center justify-center>
      <v-flex xs12 sm10 md6>
        <v-card class="elevation-12">
          <v-toolbar dark color="primary">
            <v-toolbar-title>Auth</v-toolbar-title>
            <v-spacer/>
          </v-toolbar>
          <v-card-text>
            <v-form>
              <label/>
              <v-text-field v-model="token" label="token" type="text" @keyup.enter.native="onClose()"/>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <!--<v-switch v-model="rememberMe" label="remember me"></v-switch>-->
            <v-select v-model="rememberMe"
                      :items="rememberMeTimeItems"
                      item-text="time"
                      item-value="value"
                      label="Remember Me"/>
            <v-spacer/>
            <v-btn color=primary @click="onClose()">Login</v-btn>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>

</template>
<script>
export default {
  name: 'AskForToken',
  mounted () {},
  data () {
    let rememberMe = '1y'
    if (this.$cookies.isKey('rememberMe')) {
      rememberMe =
        this.$cookies.get('rememberMe') === 'false'
          ? false
          : this.$cookies.get('rememberMe')
    }
    return {
      rememberMe,
      rememberMeTimeItems: [
        {
          time: 'No',
          value: false
        },
        {
          time: '1 year',
          value: '1y'
        },
        {
          time: '1 month',
          value: '1m'
        },
        {
          time: '1 week',
          value: '7d'
        },
        {
          time: '1 day',
          value: '1d'
        }
      ],
      token: ''
    }
  },
  created () {
    if (this.$cookies.isKey('auth')) {
      let token = this.$cookies.get('auth')
      this.auth(token)
    }
  },
  methods: {
    auth (token) {
      this.$http.post('auth', { token: token }).then(
        res => {
          this.$store.commit('login', token)
          this.$http.defaults.headers.common['bgmi-token'] = `${token}`
          if (this.rememberMe) {
            this.$cookies.set('auth', token, this.rememberMe)
          }
          this.$cookies.set('rememberMe', this.rememberMe)
          this.$nextTick(() => {
            if (this.$route.query.redirect) {
              this.$router.push(this.$route.query.redirect)
            } else {
              this.$router.push('/')
            }
          })
        },
        res => {
          this.$notify({
            type: 'error',
            text: 'auth wrong'
          })
        }
      )
    },
    onClose () {
      this.auth(this.token)
    }
  }
}
</script>
