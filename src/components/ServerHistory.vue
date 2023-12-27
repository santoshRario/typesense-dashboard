<template>
  <q-list style="min-width: 100px">
    <q-item
      v-if="$props.showLogout"
      clickable
      v-close-popup
      @click="logout"
    >
      <q-item-section>Logout</q-item-section>
      <q-item-section avatar>
        <q-icon name="sym_s_logout" />
      </q-item-section>
    </q-item>
    <q-separator v-if="$props.showLogout" />
  </q-list>
</template>


<script lang="ts">
import { NodeLoginDataInterface } from 'src/store/node/state';
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'ServerHistory',
  props: {
    showLogout: {
      default: false,
    },
  },
  computed: {
    loginHistory() {
      return this.$store.state.node.loginHistory.map(
        (history) => JSON.parse(history) as NodeLoginDataInterface
      );
    },
    error() {
      return this.$store.state.node.error;
    },
  },
  methods: {
    logout() {
        void this.$store.dispatch('node/logout');
    },
  },
});
</script>
