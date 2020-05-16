<template>
  <div id="font-sans antialiased text-gray-900 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <h1 class="mt-4 text-4xl font-semibold text-center ">Imgur Image Uploader</h1>
    <p class="text-center">Upload your image below and directly receive a link for it.</p>
    <Upload />
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import Upload from './components/Upload.vue';

export default Vue.extend({
  name: 'App',
  components: {
    Upload,
  },
});
</script>
