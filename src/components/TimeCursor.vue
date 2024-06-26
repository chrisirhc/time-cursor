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
    if (pressedKeys.size || keyPressedRecentlyTimer !== null) {
      return;
    }
    const inputElement = input.value!;

    if (inputElement.selectionStart !== inputElement.selectionEnd) {
      return;
    }
    const selectionStart = inputElement.selectionStart;
    // Find the cursor position, and append times to line ends.
    if (selectionStart) {
      const textValue = text.value!;
      const lineEndingAfter = textValue.indexOf('\n', selectionStart);
      const searchEnd = lineEndingAfter === -1 ? textValue.length : lineEndingAfter;

      const searchString = textValue.substring(selectionStart, searchEnd);
      const timeStringStart = searchString.indexOf(' [T');
      const currentTime = new Date().toLocaleTimeString();
      if (timeStringStart !== -1) {
        const timeStringEnd = searchString.indexOf(']', timeStringStart) + 1;
        const textBefore = textValue.substring(0, selectionStart + timeStringStart);
        const textAfter = textValue.substring(selectionStart + timeStringEnd);

        const prevTimeString = searchString.substring(timeStringStart, timeStringEnd);
        const prevStartTime = prevTimeString.substring(3, 11);

        const timeString = `[T${prevStartTime}–${currentTime}]`;
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

// We don't want to interrupt the user's flow while they're typing.
// If key was pressed recently, skip the changes to the text.
let keyPressedRecentlyTimer: number | null = null;
let pressedKeys = new Set<string>();

function handleKeyDown(event: KeyboardEvent) {
  pressedKeys.add(event.code);
  console.log('pressedKeys', pressedKeys)
  if (keyPressedRecentlyTimer) {
    clearTimeout(keyPressedRecentlyTimer);
    keyPressedRecentlyTimer = null;
  }
}

function handleKeyUp(event: KeyboardEvent) {
  pressedKeys.delete(event.code);
  if (pressedKeys.size !== 0) {
    return;
  }
  console.log('pressedKeys up', pressedKeys)
  if (keyPressedRecentlyTimer) {
    clearTimeout(keyPressedRecentlyTimer);
  }
  const _keyPressedRecentlyTimer = setTimeout(() => {
    if (keyPressedRecentlyTimer == _keyPressedRecentlyTimer) {
      keyPressedRecentlyTimer = null;
    }
  })
  keyPressedRecentlyTimer = _keyPressedRecentlyTimer;
}

</script>

<template>
  <textarea ref="input" v-model="text"
    @keydown="handleKeyDown"
    @keyup="handleKeyUp"
    ></textarea>
</template>

<style scoped>
textarea {
  width: 500px;
  height: 500px;
}
</style>
