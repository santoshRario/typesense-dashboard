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

          <q-card-section>
            <p class="text-red">Host: {{ process.env.TS_HOST }}</p>
          </q-card-section>

          <q-card-actions class="q-px-md row">
            <q-btn
              unelevated
              color="primary"
              size="lg"
              style="flex: 1"
              label="Login"
              @click="login()"
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

export default defineComponent({
  name: 'Login',
  data() {
    return {
      apiKey: 'xyz',
      node: {
        email: 's@gmail.com',
        host: 'typesense.staging.rario.com',
        port: '443',
        protocol: 'https',
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
    error() {
      return this.$store.state.node.error;
    },
  },
  methods: {
    login() {
      void this.$store.dispatch('node/login', {
        apiKey: this.apiKey,
        node: this.node,
      });
    },
  },
});
</script>
