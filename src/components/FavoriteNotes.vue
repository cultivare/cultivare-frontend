<template>
  <v-container>
    <h2>Favorite notes with image</h2>
    <v-row>
      <v-col v-for="note in notesWithImages" :key="note.id" cols="12" sm="6" md="4" lg="2">
        <v-card :color="note.color">
          <v-img
            :src="`/api/static/${note.image_filename}`"
            aspect-ratio="1"
            @click="showImageDialog(note)"
          >
          </v-img>

          <v-card-text class="text-center">
            <div v-html="note.text"></div>
            <div>
                    <v-chip v-for="(tag, index) in note.tags" :key="index" class="ma-2">
                      {{ tag }}
                    </v-chip>
                  </div>
            <router-link :to="`/cultures/${note.culture_id}`">
              View Culture
            </router-link>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="imageDialogVisible" max-width="80%">
      <v-card v-if="selectedNote">
        <v-img :src="selectedImage" contain></v-img>
        <v-card-title class="text-center">
          {{ selectedNote.text }}
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="imageDialogVisible = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import api from "@/services/api";

export default {
  name: "PhotoGallery",
  data() {
    return {
      notes: [],
      imageDialogVisible: false,
      selectedNote: null,
      selectedImage: null,
    };
  },
  computed: {
    notesWithImages() {
      return this.notes.filter(note => note.image_filename);
    }
  },
  async created() {
    await this.fetchFavoriteNotes();
  },
  methods: {
    async fetchFavoriteNotes() {
      try {
        const response = await api.get("/notes/?favorite=true");
        this.notes = response.data;
      } catch (error) {
        console.error("Error fetching favorite notes:", error);
      }
    },
    showImageDialog(note) {
      this.selectedNote = note;
      this.selectedImage = `/api/static/${note.image_filename}`;
      this.imageDialogVisible = true;
    },
  },
};
</script>
