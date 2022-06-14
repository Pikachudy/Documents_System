<template>
  <document-page></document-page>
</template>

<script>
import DocumentPage from './components/DocumentPage.vue'

export default {
  name: 'App',
  components: {
    DocumentPage
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
