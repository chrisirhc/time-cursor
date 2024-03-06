<script setup lang="ts">
import { ref, onMounted, onUpdated, onUnmounted } from 'vue'

// declare a ref to hold the element reference
// the name must match template ref value
const input = ref<HTMLInputElement | null>(null)
let text = defineModel<string>();
let lastSelection: null | number = null;
let intervalId: number;

onMounted(() => {
  intervalId = setInterval(() => {
    const inputElement = input.value!;
    const selectionStart = inputElement.selectionStart;
    // Find the cursor position, and append times to line ends.
    if (selectionStart) {
      const textValue = text.value!;
      const lineEndingAfter = textValue.indexOf('\n', selectionStart);
      const searchEnd = lineEndingAfter === -1 ? textValue.length : lineEndingAfter;

      const searchString = textValue.substring(selectionStart, searchEnd);
      const timeStringStart = searchString.indexOf(' [T');
      if (timeStringStart !== -1) {
        const timeStringEnd = searchString.indexOf(']', timeStringStart) + 1;
        const textBefore = textValue.substring(0, selectionStart + timeStringStart);
        const textAfter = textValue.substring(selectionStart + timeStringEnd);
        const timeString = '[T' + new Date().toLocaleTimeString() + ']';
        text.value = textBefore + ' ' + timeString + textAfter;
        lastSelection = selectionStart;
        return;
      }

      const textBefore = textValue.substring(0, searchEnd);
      const textAfter = textValue.substring(searchEnd);

      const timeString = '[T' + new Date().toLocaleTimeString() + ']';
      text.value = textBefore + ' ' + timeString + textAfter;
      lastSelection = selectionStart;
    }
  }, 1000)
});

onUpdated(() => {
  if (lastSelection === null) return;
    const inputElement = input.value!;
  inputElement.setSelectionRange(lastSelection, lastSelection);
  lastSelection = null;
});

onUnmounted(() => {
  clearInterval(intervalId);
});

</script>

<template>
  <textarea ref="input" v-model="text"></textarea>
</template>

<style scoped>
textarea {
  width: 500px;
  height: 500px;
}
</style>
