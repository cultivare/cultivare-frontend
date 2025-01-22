<template>
  <v-navigation-drawer v-model="drawer" app :permanent="permanent" :rail="rail" @update:rail="rail = $event">
    <v-list density="compact" nav>
      <v-list-item v-for="item in menuItems" :key="item.title" :prepend-icon="item.icon" :title="item.title"
        :to="item.path" link />
    </v-list>
  </v-navigation-drawer>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

export default defineComponent({
  name: 'LeftMenu',
  props: {},
  setup() {
    const drawer = ref(true);
    const permanent = ref(true);
    const rail = ref(false);

    const menuItems = [
      { title: 'Dashboard', icon: 'mdi-view-dashboard', path: '/' },
      { title: 'QR scanner', icon: 'mdi-qrcode-scan', path: '/scanner' },
      { title: 'Cultures', icon: 'mdi-mushroom-outline', path: '/cultures' },
      { title: 'Favorites', icon: 'mdi-star-outline', path: '/favorites' },
      { title: 'Tag Cloud', icon: 'mdi-cloud', path: '/tags' },
      { title: 'Statistics', icon: 'mdi-chart-areaspline', path: '/stats' },
      { title: 'About', icon: 'mdi-information-outline', path: '/about' },
    ];

    return { drawer, permanent, rail, menuItems };
  },
});
</script>