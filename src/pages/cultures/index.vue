<template>
  <div>
    <Cultures />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Cultures from '@/components/Cultures.vue'

export default defineComponent({
  components: {
    Cultures
  }
});
</script>