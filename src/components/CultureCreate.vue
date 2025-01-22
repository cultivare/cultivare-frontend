<template>
  <v-card class="pa-6">
    <v-alert v-if="showAlert" :type="alertType" closable @input="showAlert = false">
      {{ alertMessage }}
    </v-alert>
    <v-card-title>
      Create New Culture
      <v-spacer></v-spacer>
    </v-card-title>
    <v-card-text>
      <v-form @submit.prevent="createCulture">
        <v-text-field v-model="newCulture.name" label="Culture name" required />
        <v-text-field v-model="newCulture.species" label="Species" />
        <v-text-field v-model="newCulture.strain" label="Strain" />

        <v-date-input v-model="originDate" @update:model-value="updateUtcTimestamp" label="Origin Date" />
        <CultureSearch v-model="newCulture.parent_ids" combobox-label="Parent Cultures" />
        <v-text-field v-model="newCulture.source_id" label="Source ID" />
        <v-card-actions>
          <v-btn color="primary" type="submit">Create</v-btn>
          <v-btn color="secondary" @click="resetForm">Reset</v-btn>
          <v-btn color="secondary" @click="$emit('close')">Cancel</v-btn>
        </v-card-actions>
      </v-form>
    </v-card-text>
  </v-card>
</template>

<script>
import api from '@/services/api';
import moment from 'moment-timezone';
import CultureSearch from "@/components/CultureSearch.vue";

export default {
  name: 'CultureCreate',
  components: {
    CultureSearch,
  },
  props: {
    parentId: {
      type: String,
      default: null,
    },
  },
  data() {
    return {
      newCulture: {
        culture_name: '',
        species: '',
        strain: '',
        origin_date: null,
        parent_ids: [],
        source_id: '',
      },
      dateMenu: false,
      originDate: null, // store local date time (not UTC)
      showAlert: false,
      alertMessage: '',
      alertType: 'success', // Default to success
    };
  },
  computed: {
    formattedDate() {
      return this.newCulture.origin_date ? this.formatDate(this.newCulture.origin_date) : '';
    },
    parentIds: {
      get() {
        return this.newCulture.parent_ids.join(', ');
      },
      set(value) {
        this.newCulture.parent_ids = value ? value.split(',').map(id => id.trim()) : [];
      },
    },
  },
  watch: {
    parentId: {
      immediate: true,
      handler(newParentId) {
        this.newCulture.parent_ids = newParentId ? [newParentId] : [];
      },
    },
  },
  async mounted() {
    this.getCurrentTimestamp();
  },
  methods: {
    updateUtcTimestamp() {
      if (this.originDate) {
        const userTimezone = moment.tz.guess();
        // Set time to 12 PM in user's timezone
        const localOriginDateTime = moment.tz(this.originDate, userTimezone).set({ hour: 12, minute: 0, second: 0, millisecond: 0 });
        const utcOriginDateTime = localOriginDateTime.utc();
        this.newCulture.origin_date = utcOriginDateTime.format();
      } else {
        this.newCulture.origin_date = null;
      }
    },
    getCurrentTimestamp() {
      // get current time in utc and retun it as local timezone
      const utcDateTime = moment.utc();
      const userTimezone = moment.tz.guess();
      const localDateTime = utcDateTime.tz(userTimezone);
      this.originDate = localDateTime.toDate();
    },
    async createCulture() {
      try {
        this.updateUtcTimestamp()
        const response = await api.post('/cultures/', this.newCulture);

        if (response.status === 201) {
          this.$emit('culture-created', response.data);
          this.resetForm();
          this.showAlert = true;
          this.alertMessage = 'Culture created successfully!';
          this.alertType = 'success';
          // Redirect to the culture page after successful creation
          this.$router.push(`/cultures/${response.data.id}`); 
        } else {
          throw new Error(`Error creating culture: ${response.status}`);
        }
      } catch (error) {
        console.error(error);
        this.showAlert = true;
        this.alertMessage = 'Failed to create culture. Please try again.';
        this.alertType = 'error';
      }
    },
    resetForm() {
      this.newCulture = {
        culture_name: '',
        species: '',
        strain: '',
        origin_date: null,
        parent_ids: this.parentId ? [this.parentId] : [],
        source_id: '',
      };
      this.dateMenu = false;
    },
  },
};
</script>