<template>
    <v-container>
      <v-row>
 
        <v-col cols="12" md="6">
          <v-card class="pa-4">
            <v-card-title>Cultures Statistics</v-card-title>
            <v-card-text>
              <p>Total Cultures: {{ stats.cultures_count }}</p>
              <p>Favorite Cultures: {{ stats.favorite_cultures_count }}</p>
              <p>Total Parents: {{ stats.culture_parent_ids_count }}</p>
              <p>Cultures Created Last Month: {{ stats.cultures_created_last_month }}</p>
              <p>Cultures Updated Last Month: {{ stats.cultures_updated_last_month }}</p>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card class="pa-4">
            <v-card-title>Notes Statistics</v-card-title>
            <v-card-text>
              <p>Total Notes: {{ stats.notes_count }}</p>
              <p>Favorite Notes: {{ stats.favorite_notes_count }}</p>
              <p>Notes with Images: {{ stats.images_count }}</p>
              <p>Note Colors: {{ stats.note_colors_count }}</p>
              <p>Notes Created Last Month: {{ stats.notes_created_last_month }}</p>
              <p>Notes Updated Last Month: {{ stats.notes_updated_last_month }}</p>
            </v-card-text>
          </v-card>
        </v-col>  

        <v-col cols="12">
          <v-card class="pa-4">
            <v-card-title>Tags Statistics</v-card-title>
            <v-card-text>
              <p>Unique Tags: {{ stats.tag_count }}</p>
              <p>Total Tag Uses: {{ stats.tag_uses }}</p>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </template>
  
  <script>
  import api from "@/services/api";

  export default {
    data() {
      return {
        stats: {},
      };
    },
    async mounted() {
      try {
        const response = await api.get('/stats/');
        this.stats = response.data;
      } catch (error) {
        console.error('Error fetching stats:', error);
        // Handle error, e.g., show an error message to the user
      }
    },
  };
  </script>