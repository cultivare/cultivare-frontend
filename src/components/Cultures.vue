<template>
  <div v-if="cultures"> 
    <CulturesTable :cultures="cultures" />
  </div>
  <div v-else>Loading...</div> 
</template>
  
  <script>
  import api from '@/services/api';
  
  import CulturesTable from '@/components/CulturesTable.vue';
 
  export default {
    name: 'Cultures',
    components: {
    CulturesTable,
  },
  data() {
    return {
      cultures: null,
      loading: false,
    };
  },
  async mounted() {
    await this.fetchCultures();
  },
  methods: {
    async fetchCultures() {
            this.loading = true;
            try {
              const response = await api.get('/cultures/');
                this.cultures = response.data;
            } catch (error) {
                console.error(error);
                // Handle error
            } finally {
                this.loading = false;
            }
        },
      }
};
</script>