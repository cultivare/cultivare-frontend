<template>
    <v-card>
      <v-card-title>
        Cultures
        <v-spacer></v-spacer>
        <v-text-field 
          v-model="search" 
          append-icon="mdi-magnify" 
          label="Search" 
          single-line 
          hide-details>
        </v-text-field>
      </v-card-title>
      <v-card-actions>
        <v-btn color="primary" @click="showCreateCultureDialog = true">Create Culture</v-btn>
      </v-card-actions>
  
      <v-data-table 
        :headers="headers" 
        :items="cultures" 
        :search="search" 
        :loading="loading"
        loading-text="Loading... Please wait">
        <template v-slot:item.origin_date="{ item }">
          {{ formatDate(item.origin_date) }}
        </template>
        <template v-slot:item.name="{ item }">
          <router-link :to="`/cultures/${item.id}`">
            {{ item.name }}
          </router-link>
        </template>
        <!-- <template v-slot:item.tags="{ item }"> 
          <v-chip v-for="tag in item.tags" :key="tag" small>{{ tag }}</v-chip>
        </template> -->
      </v-data-table>
  
      <v-dialog v-model="showCreateCultureDialog" max-width="600px">
        <CultureCreate @close="showCreateCultureDialog = false" />
      </v-dialog>
    </v-card>
  </template>
  
  <script>
  import CultureCreate from '@/components/CultureCreate.vue';
  import moment from 'moment-timezone';
  
  export default {
    name: 'CulturesTable',
    components: {
      CultureCreate,
    },
    props: {
      cultures: {
        type: Array,
        required: true,
        default: () => [],
      },
    },
    data() {
      return {
        loading: false,
        search: '',
        headers: [
          { title: 'Name', value: 'name', sortable: true },
          { title: 'Species', value: 'species' },
          { title: 'Strain', value: 'strain' },
        //   { title: 'Tags', value: 'tags' }, 
          { title: 'Origin Date', value: 'origin_date' }
        ],
        showCreateCultureDialog: false,
      };
    },
    methods: {
      formatDate(date) {
        return moment.utc(date).tz(moment.tz.guess()).format('YYYY-MM-DD');
      }
    },
  };
  </script>