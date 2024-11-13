<script setup lang="ts">
import { ref, onMounted } from "vue";
import { invoke } from "@tauri-apps/api/core";
import { getCurrentWindow, LogicalSize } from '@tauri-apps/api/window';

const greetMsg = ref("");
const name = ref("");
const windowSize = ref({ width: 0, height: 0 });
const isSmallSize = ref(true);
const warningMsg = ref("");
const previousWindowSize = ref({ width: 0, height: 0 });

async function greet() {
  greetMsg.value = await invoke("greet", { name: name.value });
}

onMounted(async () => {
  const webview = getCurrentWindow();
  const size = await webview.innerSize();
  const scaleFactor = await webview.scaleFactor();
  windowSize.value = { width: size.width / scaleFactor, height: size.height / scaleFactor };
  previousWindowSize.value = { width: size.width / scaleFactor, height: size.height / scaleFactor };
});

const onDragStart = async (e: MouseEvent) => {
  const noDragSelector = 'input, a, button';
  if (e.target instanceof Element && e.target.closest(noDragSelector)) return;
  await getCurrentWindow().startDragging();
}

const closeApp = async () => {
  await getCurrentWindow().close();
}

const checkWindowSize = async (expectedWidth: number, expectedHeight: number) => {
  const webview = getCurrentWindow();
  const size = await webview.innerSize();
  const scaleFactor = await webview.scaleFactor();
  const logicalWidth = size.width / scaleFactor;
  const logicalHeight = size.height / scaleFactor;
  if (logicalWidth !== expectedWidth || logicalHeight !== expectedHeight) {
    if (logicalWidth === previousWindowSize.value.width && logicalHeight === previousWindowSize.value.height) {
      warningMsg.value = "Error: Window size did not change!";
    } else {
      warningMsg.value = "Warning: Window size did not update correctly!";
    }
    return false;
  }
  return true;
}

const toggleWindowSize = async () => {
  const webview = getCurrentWindow();
  let newSize;
  if (isSmallSize.value) {
    newSize = new LogicalSize(600, 400);
  } else {
    newSize = new LogicalSize(500, 300);
  }
  previousWindowSize.value = { width: windowSize.value.width, height: windowSize.value.height };
  await webview.setSize(newSize);

  setTimeout(async () => {
    const success = await checkWindowSize(newSize.width, newSize.height);
    if (success) {
      windowSize.value = { width: newSize.width, height: newSize.height };
      warningMsg.value = "";
      isSmallSize.value = !isSmallSize.value;
    }
  }, 200);
}
</script>

<template>
  <main class="container" @pointerdown="onDragStart">
    <button class="quit-button" @click="closeApp">X</button>
    <h1>Welcome to Tauri + Vue</h1>
    <div>The window is draggable</div>
    <div style="display: flex; flex-direction: column; margin-bottom: 1rem;">
      <div>Toggling is between 500 x 300 & 600 x 400</div>
      <div>Actual size (logical): {{ windowSize.width }} x {{ windowSize.height }}</div>
    </div>

    <form class="row" @submit.prevent="greet">
      <input id="greet-input" v-model="name" placeholder="Enter a name..." />
      <button type="submit">Greet</button>
      <button type="button" @click="toggleWindowSize">Toggle Size</button>
    </form>
    <p>{{ greetMsg }}</p>
    <p style="color: red;">{{ warningMsg }}</p>
  </main>
</template>

<style scoped>
.logo.vite:hover {
  filter: drop-shadow(0 0 2em #747bff);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #249b73);
}
</style>
<style>
:root {
  font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;

  color: #0f0f0f;
  background-color: #f6f6f6;

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-text-size-adjust: 100%;
}

body {
  margin: 0;
}

.container {
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  min-height: 100vh;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: 0.75s;
}

.logo.tauri:hover {
  filter: drop-shadow(0 0 2em #24c8db);
}

.row {
  display: flex;
  justify-content: center;
  gap: 1rem;
}

a {
  font-weight: 500;
  color: #646cff;
  text-decoration: inherit;
}

a:hover {
  color: #535bf2;
}

h1 {
  text-align: center;
}

input,
button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  color: #0f0f0f;
  background-color: #ffffff;
  transition: border-color 0.25s;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
}

button {
  cursor: pointer;
}

button:hover {
  border-color: #396cd8;
}

button:active {
  border-color: #396cd8;
  background-color: #e8e8e8;
}

input,
button {
  outline: none;
}

#greet-input {
  margin-right: 5px;
  width: 200px;
}

.quit-button {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 5px 10px;
  font-size: 16px;
  background: darkgoldenrod;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.quit-button:hover {
  background: red;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #f6f6f6;
    background-color: #2f2f2f;
  }

  a:hover {
    color: #24c8db;
  }

  input,
  button {
    color: #ffffff;
    background-color: #0f0f0f98;
  }

  button:active {
    background-color: #0f0f0f69;
  }
}
</style>