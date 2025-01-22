<template>
    <div>
      <About />
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent } from 'vue';

import About from '@/components/About.vue';
  
  export default defineComponent({
    components: {
        About
    }
  });
  </script>