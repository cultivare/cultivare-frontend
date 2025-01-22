<template>
  <QrcodeStream />
</template>

<script>
import { defineComponent } from 'vue';
import QrcodeStream from '@/components/QrcodeStream.vue';

export default defineComponent({
  components: {
    QrcodeStream
  }
});
</script>