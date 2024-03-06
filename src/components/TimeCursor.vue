<script setup lang="ts">
const text = 'Hello, Vue 3!';
</script>

<template>
  <textarea v-model="text"></textarea>
</template>

<style scoped>
textarea {
  width: 500px;
  height: 500px;
}
</style>
