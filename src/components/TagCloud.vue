<template>
  <v-container>
    <v-row class="text-center">
      <v-col>
        <v-card>
          <v-card-title>Tag Cloud</v-card-title>
          <v-card-text>
            <div id="wordcloud" style="width: 100%; height: 1000px;"></div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import api from "@/services/api";
import WordCloud from 'wordcloud';

export default {
  data() {
    return {
      tagFrequency: {},
    };
  },
  async mounted() {
    await this.fetchTagFrequency();
    this.$nextTick(() => {
      this.generateWordCloud();
    });
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    // remove the event listener when the component is destroyed
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    async fetchTagFrequency() {
      try {
        const response = await api.get('/tags/frequency');
        this.tagFrequency = response.data;
      } catch (error) {
        console.error('Error fetching tag frequency:', error);
      }
    },
    generateWordCloud() {
      const words = Object.entries(this.tagFrequency).map(([tag, count]) => [tag, count]);
      const canvas = document.getElementById('wordcloud');

      // Clear the canvas before regenerating the word cloud
      canvas.innerHTML = '';

      WordCloud(canvas, {
        list: words,
        gridSize: Math.round(16 * canvas.offsetWidth / 1024),
        weightFactor: (size) => size * 22 * canvas.clientWidth / 1024,
        fontFamily: 'Times, serif',
        shape: 'circle',
        rotateRatio: 0,
        rotationSteps: 2,
        backgroundColor: '#212121',
        drawOutOfBound: false,
        hover: function (item, dimension, event) {
          if (item) {
            event.target.style.textDecoration = 'underline';
            event.target.style.cursor = 'pointer';
          } else {
            event.target.style.textDecoration = 'none';
            event.target.style.cursor = 'default';
          }
        },
      });
    },
    handleResize() {
      // Debounce the resize event to improve performance
      clearTimeout(this.resizeTimer);
      this.resizeTimer = setTimeout(() => {
        this.generateWordCloud();
      }, 200); // Adjust the delay as needed
    }
  }
};
</script>