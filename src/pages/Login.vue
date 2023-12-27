<template>
  <div class="fullscreen row bg-primary text-center q-pa-md flex flex-center">
    <div class="col col-md-4">
      <div>
        <h5 class="text-h5 text-white q-my-md">Typesense Dashboard</h5>
      </div>
      <div>
        <q-card bordered class="q-pa-lg shadow-1">
          <q-card-section v-if="error">
            <p class="text-red">{{ error }}</p>
          </q-card-section>

          <q-card-actions class="q-px-md row">
            <q-btn
              unelevated
              color="primary"
              size="lg"
              style="flex: 1; opacity: `${isLoading ? 0.5 : 1 }`"
              :label="`${isLoading ? 'loading ...' : 'Login With Google' }`"
              @click="loginWithGoogle()"
            />
          </q-card-actions>
        </q-card>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { NodeLoginDataInterface } from 'src/store/node/state';
import { defineComponent } from 'vue';
import { googleSdkLoaded } from 'vue3-google-login';
import axios from 'axios';

export default defineComponent({
  name: 'Login',
  data() {
    return {
      apiKey: process.env.TS_APIKEY,
      loading: false,
      node: {
        email: '',
        host: process.env.TS_HOST,
        port: process.env.TS_PORT,
        protocol: process.env.TS_PROTOCOL,
        path: '',
        tls: true,
      },
    };
  },
  computed: {
    loginHistory() {
      return this.$store.state.node.loginHistory.map(
        (j) => JSON.parse(j) as NodeLoginDataInterface
      );
    },
    isLoading(): boolean {
      return this.loading
    },
    error() {
      return this.$store.state.node.error;
    },
  },
  methods: {
    loginWithGoogle() {
      if (this.loading) return
      this.loading = true
      googleSdkLoaded(google => {
        google.accounts.oauth2
          .initCodeClient({
            client_id: process.env.G_AUTH_CLIENT_ID || '',
            scope: 'email profile openid',
            callback: response => {
              if (response.code) {
                // console.log('response', response)
                void this.sendCodeToBackend(response.code);
              } else {
                this.loading = false
              }
            },
            error_callback: response => {
              if (response) {
                this.loading = false
              }
            }
          })
          .requestCode();
      });
    },
    async sendCodeToBackend(code: string) {
      try {
        const response = await axios.post(
          'https://oauth2.googleapis.com/token',
          {
            code,
            client_id: process.env.G_AUTH_CLIENT_ID,
            client_secret: process.env.G_AUTH_CLIENT_SECRET,
            redirect_uri: 'postmessage',
            grant_type: 'authorization_code'
          }
        );

        const accessToken = response.data.access_token;
        // console.log(accessToken);

        // Fetch user details using the access token
        const userResponse = await axios.get(
          'https://www.googleapis.com/oauth2/v3/userinfo',
          {
            headers: {
              Authorization: `Bearer ${accessToken}`
            }
          }
        );

        if (userResponse && userResponse.data) {
          // Set the userDetails data property to the userResponse object
          this.node = { ...this.node, email: userResponse.data.email }
          // console.log('userdata', userResponse)
          this.login()
        } else {
          // Handle the case where userResponse or userResponse.data is undefined
          console.error('Failed to fetch user details.');
          this.loading = false
        }
      } catch (error) {
        this.loading = false
        console.error('Token exchange failed:', error);
      }
    },
    login() {
      void this.$store.dispatch('node/login', {
        apiKey: this.apiKey,
        node: this.node,
      });
    },
  },
});
</script>
