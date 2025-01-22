<template>
    <v-card class="pa-6">
        <v-alert v-if="showAlert" :type="alertType" @input="showAlert = false" closable>
            {{ alertMessage }}
        </v-alert>

        <v-card-title>About Cultivare</v-card-title>
        <v-card-subtitle> </v-card-subtitle>
        <v-card-text>
            <v-row>
                <v-col cols="12" md="8">
                    <!-- <a href="/docs">Cultivare docs</a><br /> -->
                    <a href="https://www.patreon.com/cultivare" target="_blank">Cultivare Patreon</a><br />
                    <a href="/api/docs">API Swagger docs</a><br />
                    <a href="/api/redoc">API redoc docs</a><br />
                    <!-- <a href="/traefik/dashboard/">Traefik Dashboard</a><br /> -->
                </v-col>

                <v-col cols="12" md="4">

                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>

<script >

export default {
    name: 'About',
    components: {
    },
    data() {
        return {
        };
    },
};
</script>