<template>
  <div>
    <CultureDetails />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import CultureDetails from '@/components/CultureDetails.vue'

export default defineComponent({
  components: {
    CultureDetails
  }
});
</script>