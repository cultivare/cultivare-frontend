<template>
    <v-app>
        <v-app-bar>
            <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
            <v-app-bar-title>
                Cultivare
                <v-icon color="red-darken-2" icon="mdi-mushroom" size="22"  />
            </v-app-bar-title>
        </v-app-bar>

        <LeftMenu v-model="drawer" :permanent="permanent" />

        <v-main>
            <router-view />
        </v-main>
    </v-app>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import LeftMenu from '@/components/LeftMenu.vue';

export default defineComponent({
    setup() {
        const drawer = ref(true);
        const permanent = ref(false); // Initially not permanent

        // Make permanent on larger screens
        const isPermanent = computed(() => permanent.value || window.innerWidth >= 1264);

        return { drawer, permanent, isPermanent };
    },
    components: { LeftMenu }
});
</script>