<template>
  <h1>Hello from Users Dashboard!</h1>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Dashboard from '@/components/Dashboard.vue'; // Make sure to import the Dashboard component

export default defineComponent({
  components: {
    Dashboard
  }
});
</script>