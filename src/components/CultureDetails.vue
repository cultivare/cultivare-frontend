<template>
  <v-card v-if="culture" class="pa-6">
    <v-alert v-if="showAlert" :type="alertType" @input="showAlert = false" closable>
      {{ alertMessage }}
    </v-alert>

    <v-card-title>
      {{ culture.name }}
      <v-icon size="x-small" @click="culture.favorite = !culture.favorite"
        :color="culture.favorite ? 'amber' : 'inherit'">
        {{ culture.favorite ? 'mdi-star' : 'mdi-star-outline' }}
      </v-icon>
    </v-card-title>

    <v-card-subtitle>
      id: {{ culture.id }}<br />
      created {{ formatRelativeTime(culture.created_at) }} ({{ formatDate(culture.created_at) }})
    </v-card-subtitle>

    <v-card-text>
      <v-row>
        <v-col cols="12" md="8">
          <v-form @submit.prevent="saveCulture">
            <v-text-field v-model="culture.name" label="Culture name" />
            <v-text-field v-model="culture.species" label="Species" />
            <v-text-field v-model="culture.strain" label="Strain" />
            <v-date-input v-model="originDate" @update:model-value="updateUtcTimestamp" label="Origin Date" />
            <CultureSearch v-model="culture.parent_ids" combobox-label="Parent Cultures" />
            <TagSearch v-model="culture.tags" combobox-label="Tags" />

            <v-card-actions>
              <v-btn color="primary" type="submit">Save</v-btn>
              <v-btn color="secondary" @click="resetCulture">Reset</v-btn>
              <v-btn color="primary" @click="dialog = true">Create Child Culture</v-btn>
              <v-btn color="primary" dark @click="dialogBarcodeVisible = true">Create Label</v-btn>
            </v-card-actions>
          </v-form>

          <Notes :key="culture.id" :culture-id="culture.id" />
        </v-col>
        <v-col cols="12" md="4">
          <GraphView :key="culture.id" :culture_id="culture.id" />
        </v-col>
      </v-row>
    </v-card-text>
  </v-card>

  <div v-else>Loading...</div>

  <v-dialog v-model="dialog" max-width="600px">
    <v-card>
      <culture-create :parent-id="culture.id" @close="dialog = false" @culture-created="onCultureCreated" />
    </v-card>
  </v-dialog>

  <v-dialog v-model="dialogBarcodeVisible" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Label Generator</span>
        </v-card-title>
        <v-card-text>
          <Barcode :barcodeData="barcodeData" />
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" @click="dialogBarcodeVisible = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
</template>

<script>
import api from "@/services/api";
import moment from "moment-timezone";
import CultureCreate from "@/components/CultureCreate.vue";
import Notes from "@/components/Notes.vue";
import CultureSearch from "@/components/CultureSearch.vue";
import GraphView from "@/components/GraphView.vue";
import TagSearch from "./TagSearch.vue";
import Barcode from '@/components/Barcode.vue';

export default {
  name: "CultureDetails",
  components: {
    CultureCreate,
    GraphView,
    Notes,
    CultureSearch,
    TagSearch,
    Barcode,
  },
  data() {
    return {
      culture: null,
      originalCulture: null,
      dateMenu: false,
      dialog: false,
      loading: false,
      showAlert: false,
      alertMessage: "",
      alertType: "success",
      originDate: null,
      
      // Barcode settings:
      dialogBarcodeVisible: false,
    };
  },
  computed: {
    barcodeData() {
    return {
      barcodeType: 'qrcode',
      barcodeText: this.culture ? this.culture.id : '', // Use a default value if culture is null
      labelText: this.culture ? this.culture.name : '', // Use a default value if culture is null
      dateText: this.culture ? this.formatDate(this.culture.origin_date) : '', // Use a default value if culture is null
      showSecondLine: true,
    };
  },
    formattedDate() {
      return this.culture?.origin_date
        ? this.formatDate(this.culture.origin_date)
        : "";
    },
  },
  async mounted() {
    await this.fetchCulture();
    this.initializeDateTime();
  },
  watch: {
    "$route.params.id": {
      handler: async function (newId) {
        if (newId) {
          await this.fetchCulture();
          this.initializeDateTime();
        }
      },
      immediate: true,
    },
  },
  methods: {
    async fetchCulture() {
      this.loading = true;
      try {
        const cultureId = this.$route.params.id;
        const response = await api.get(`/cultures/${cultureId}`);
        this.culture = response.data;

        if (this.culture.parent_ids && this.culture.parent_ids.length > 0) {
          const parentCultures = await this.fetchParentCultures(this.culture.parent_ids);
          this.culture.parents = parentCultures;
        } else {
          this.culture.parents = [];
        }

        this.originalCulture = { ...this.culture };
      } catch (error) {
        console.error("Error fetching culture:", error);
        this.showAlert = true;
        this.alertMessage = "Error fetching culture. Please try again.";
        this.alertType = "error";
      } finally {
        this.loading = false;
        console.log("culture:", this.culture);
      }
    },
    async fetchParentCultures(parentIds) {
      try {
        const response = await api.get(`/cultures/search`, {
          params: { parent_ids: parentIds },
          paramsSerializer: {
            indexes: null
          }
        });
        return response.data;
      } catch (error) {
        console.error("Error fetching parent cultures:", error);
        return [];
      }
    },
    async saveCulture() {
      this.loading = true;
      try {
        const cultureId = this.$route.params.id;
        // Send only necessary data to the backend
        const cultureData = {
          ...this.culture,
        };
        const response = await api.put(`/cultures/${cultureId}`, cultureData);
        this.culture = response.data;
        this.originalCulture = { ...this.culture };
        this.showAlert = true;
        this.alertMessage = "Culture saved successfully!";
        this.alertType = "success";
      } catch (error) {
        console.error("Error saving culture:", error);
        this.showAlert = true;
        this.alertMessage = "Error saving culture. Please try again.";
        this.alertType = "error";
      } finally {
        this.loading = false;
      }
    },
    resetCulture() {
      this.culture = { ...this.originalCulture };
    },
    onCultureCreated() {
      this.fetchCulture();
      this.showAlert = true;
      this.alertMessage = "Child culture created successfully!";
      this.alertType = "success";
    },
    formatRelativeTime(utcTimestamp) {
      return moment.utc(utcTimestamp).fromNow();
    },
    formatDate(utcTimestamp) {
      return moment.utc(utcTimestamp).tz(moment.tz.guess()).format("MMM DD, YYYY");
    },
    updateUtcTimestamp() {
      if (this.originDate) {
        const userTimezone = moment.tz.guess();
        // Set time to 12 PM in user's timezone
        const localOriginDateTime = moment.tz(this.originDate, userTimezone).set({ hour: 12, minute: 0, second: 0, millisecond: 0 });
        const utcOriginDateTime = localOriginDateTime.utc();
        this.culture.origin_date = utcOriginDateTime.format();
      } else {
        this.culture.origin_date = null;
      }
    },
    initializeDateTime() {
      if (this.culture && this.culture.origin_date) {
        const utcDateTime = moment.utc(this.culture.origin_date);
        const userTimezone = moment.tz.guess();
        const localDateTime = utcDateTime.tz(userTimezone);
        this.originDate = localDateTime.toDate();
      }
    },
  },
};
</script>