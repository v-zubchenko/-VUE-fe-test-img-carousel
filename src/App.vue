<template>
  <div v-if="loading">Загрузка...</div>
  <div v-else>
    <AppImgCarousel :data="data" />
  </div>
</template>

<script>
import AppImgCarousel from './components/AppImgCarousel.vue';
import axios from "axios";

export default {
  components: {
    AppImgCarousel,
  },

  data() {
    return { data: [], loading: true };
  },
  created() {
    this.fetchImages();
  },

  methods: {
    fetchImages() {
      axios.get('https://picsum.photos/v2/list?page=1&limit=10')
          .then(response => {
            this.data = response.data.map(img => ({
              ...img,
              download_url: `https://picsum.photos/id/${img.id}/500/333`
            }));
          })
          .catch(error => {
            console.error('', error);
          })
          .finally(() => {
            this.loading = false;
          });
    },
  },

};
</script>


<style>

</style>
