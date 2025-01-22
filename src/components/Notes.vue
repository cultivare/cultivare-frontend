<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-btn color="primary" @click="showCreateDialog = true">
          Add Note
        </v-btn>
      </v-col>
    </v-row>

    <v-row>
      <v-col v-for="note in sortedNotes" :key="note.id" cols="12">
        <v-card :color="note.color">
          <div class="d-flex flex-column flex-sm-row">
            <v-avatar v-if="note.image_filename" class="ma-3" rounded="0" size="150" @click="
              showImageDialog = true;
            selectedImage = `/api/static/${note.image_filename}`;
            ">
              <v-img :src="`/api/static/${note.image_filename}`"></v-img>
            </v-avatar>
            <div class="flex-grow-1">
              <v-card-text class="d-flex flex-column">
                <div v-if="editingNoteId === note.id">
                  <v-textarea v-model="editingNoteText" label="Edit Note"></v-textarea>
                  <TagSearch v-if="editingNoteId === note.id" v-model="editingNoteTags" comboboxLabel="Edit Tags"
                    @update:modelValue="updateNoteTags(note.id, $event)" />
                  <v-row>
                    <v-col cols="12">
                      <v-file-input v-model="editingNoteFile" label="Image (optional)" accept="image/*"
                        prepend-icon="mdi-image" />
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-btn @click="showColorPicker = !showColorPicker">Set Color</v-btn>
                      <v-expand-transition>
                        <div v-if="showColorPicker">
                          <v-color-picker show-swatches :swatches="swatches" v-model="editingNoteColor" dot-size="25"
                            swatches-max-height="200"></v-color-picker>
                        </div>
                      </v-expand-transition>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-checkbox v-model="editingNoteFavorite" label="Favorite"></v-checkbox>
                    </v-col>
                  </v-row>
                  <div class="pt-5">
                    <v-btn color="success" @click="updateNote(note.id)" class="mr-2">Save</v-btn>
                    <v-btn color="error" @click="cancelEdit">Cancel</v-btn>
                  </div>
                </div>
                <div v-else class="flex-grow-1">
                  <div class="d-flex align-center">
                    <div v-html="note.text" class="text-pre-wrap"></div>
                    <v-icon v-if="note.favorite" color="amber">mdi-star</v-icon>
                  </div>
                  <div>
                    <v-chip v-for="(tag, index) in note.tags" :key="index" class="ma-2">
                      {{ tag }}
                    </v-chip>
                  </div>
                  <v-card-subtitle class="pt-5 d-flex justify-end">
                    Created {{ formatRelativeTime(note.created_at) }} ({{
                      formatDate(note.created_at)
                    }})
                  </v-card-subtitle>
                </div>
              </v-card-text>
              <v-card-actions v-if="editingNoteId !== note.id" class="d-flex justify-end">
                <v-btn icon @click="
                  startEdit(
                    note.id,
                    note.text,
                    note.image_filename,
                    note.color,
                    note.tags,
                    note.favorite
                  );
                ">
                  <v-icon>mdi-pencil</v-icon>
                </v-btn>
                <v-btn icon @click="
                  showDeleteConfirmation = true;
                noteToDeleteId = note.id;
                " color="red">
                  <v-icon>mdi-delete</v-icon>
                </v-btn>
              </v-card-actions>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="showCreateDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Create New Note</span>
        </v-card-title>
        <v-card-text>
          <v-textarea v-model="newNoteText" label="Note Text"></v-textarea>
          <TagSearch v-model="newNoteTags" comboboxLabel="Tags" />
          <v-file-input v-model="newNoteFile" label="Image (optional)" accept="image/*"
            prepend-icon="mdi-image"></v-file-input>
          <v-btn :color="newNoteColor" @click="showNewColorPicker = !showNewColorPicker">Set Color</v-btn>
          <v-expand-transition>
            <div v-if="showNewColorPicker">
              <v-color-picker show-swatches :swatches="swatches" v-model="newNoteColor" dot-size="25"
                swatches-max-height="200"></v-color-picker>
            </div>
          </v-expand-transition>
          <v-checkbox v-model="newNoteFavorite" label="Favorite"></v-checkbox>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="createNote">Create</v-btn>
          <v-btn @click="showCreateDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showDeleteConfirmation" max-width="500px">
      <v-card>
        <v-card-title class="headline">Confirm Delete</v-card-title>
        <v-card-text> Are you sure you want to delete this note? </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" @click="deleteNote(noteToDeleteId)">Delete</v-btn>
          <v-btn @click="showDeleteConfirmation = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showImageDialog" max-width="80%">
      <v-card>
        <v-img :src="selectedImage" contain></v-img>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="showImageDialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import api from "@/services/api";
import moment from "moment-timezone";
import TagSearch from "@/components/TagSearch.vue";

export default {
  components: {
    TagSearch,
  },
  props: {
    cultureId: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      notes: [],
      showCreateDialog: false,
      newNoteText: "",
      newNoteFile: null,
      newNoteColor: "#2a2a2a",
      newNoteTags: [],
      newNoteFavorite: false, // Add this for the new note's favorite status
      showNewColorPicker: false,
      editingNoteId: null,
      editingNoteText: "",
      editingNoteFile: null,
      editingNoteColor: "",
      editingNoteTags: [],
      editingNoteFavorite: false, // Add this for the editing note's favorite status
      showColorPicker: false,
      swatches: [
        ["#FFFF88", "#FFD700", "#FFA500", "#FF7F50", "#FF0000"],
        ["#FFC0CB", "#FFB6C1", "#FF69B4", "#DB7093", "#C71585"],
        ["#9370DB", "#8A2BE2", "#4B0082", "#483D8B", "#000080"],
        ["#00FFFF", "#00CED1", "#40E0D0", "#20B2AA", "#008080"],
        ["#2A2A2A", "#7CFC00", "#32CD32", "#228B22", "#006400"],
      ],
      showDeleteConfirmation: false,
      noteToDeleteId: null,
      showImageDialog: false,
      selectedImage: null,
    };
  },
  computed: {
    sortedNotes() {
      return this.notes.slice().sort((a, b) => {
        return new Date(b.created_at) - new Date(a.created_at);
      });
    },
  },
  async created() {
    await this.fetchNotes();
  },
  methods: {
    formatRelativeTime(utcTimestamp) {
      return moment.utc(utcTimestamp).fromNow();
    },
    formatDate(utcTimestamp) {
      return moment.utc(utcTimestamp).tz(moment.tz.guess()).format("MMM DD, YYYY");
    },
    async fetchNotes() {
      try {
        const response = await api.get(`/notes/culture/${this.cultureId}`);
        this.notes = response.data;
      } catch (error) {
        console.error("Error fetching notes:", error);
      }
    },
    async createNote() {
      const formData = new FormData();
      formData.append("text", this.newNoteText);
      formData.append("culture_id", this.cultureId);
      formData.append("color", this.newNoteColor);
      formData.append("favorite", this.newNoteFavorite); // Add favorite status
      // Include tags when creating a new note
      if (this.newNoteTags.length > 0) {
        formData.append("tags", JSON.stringify(this.newNoteTags));
      }
      if (this.newNoteFile) {
        formData.append("file", this.newNoteFile);
      }

      try {
        const response = await api.post("/notes/", formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        });
        this.notes.push(response.data);
        this.showCreateDialog = false;
        this.newNoteText = "";
        this.newNoteFile = null;
        this.newNoteColor = "#000000";
        this.newNoteTags = []; // Reset tags
        this.newNoteFavorite = false; // Reset favorite status
      } catch (error) {
        console.error("Error creating note:", error);
      }
    },
    startEdit(noteId, text, image_filename, color, tags, favorite) {
      this.editingNoteId = noteId;
      this.editingNoteText = text;
      this.editingNoteFile = null;
      this.editingNoteColor = color;
      this.editingNoteTags = tags || [];
      this.editingNoteFavorite = favorite; // Set the favorite status
    },
    async updateNote(noteId) {
      const formData = new FormData();
      if (this.editingNoteText !== undefined) {
        formData.append("text", this.editingNoteText);
      }
      if (this.editingNoteColor) {
        formData.append("color", this.editingNoteColor);
      }
      formData.append("favorite", this.editingNoteFavorite); // Add favorite status
      // Include tags when updating a note
      if (this.editingNoteTags.length > 0) {
        formData.append("tags", JSON.stringify(this.editingNoteTags));
      }
      if (this.editingNoteFile) {
        formData.append("file", this.editingNoteFile);
      }

      try {
        const response = await api.put(`/notes/${noteId}`, formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        });
        const index = this.notes.findIndex((note) => note.id === noteId);
        if (index !== -1) {
          this.notes.splice(index, 1, response.data);
        }
        this.cancelEdit();
      } catch (error) {
        console.error("Error updating note:", error);
      }
    },
    updateNoteTags(noteId, updatedTags) {
      const note = this.notes.find((n) => n.id === noteId);
      if (note) {
        note.tags = updatedTags;
        this.editingNoteTags = updatedTags;
      }
    },
    async deleteNote(noteId) {
      try {
        await api.delete(`/notes/${noteId}`);
        this.notes = this.notes.filter((note) => note.id !== noteId);
      } catch (error) {
        console.error("Error deleting note:", error);
      } finally {
        this.showDeleteConfirmation = false;
      }
    },
    cancelEdit() {
      this.editingNoteId = null;
      this.editingNoteText = "";
      this.editingNoteFile = null;
      this.editingNoteColor = "";
      this.editingNoteTags = [];
      this.editingNoteFavorite = false; // Reset favorite status
      this.showColorPicker = false;
    },
  },
};
</script>

<style scoped>
.text-pre-wrap {
  white-space: pre-wrap;
}
</style>