<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <RouterView />
</template>

<style>
:root {
  --color-primary: #002a90;
  --color-primary-dark: #001653;
  --color-cyan: #01c3f7;
  --color-yellow: #f5c759;
  --color-red: #f50201;
  --color-ink: #0e1730;
  --color-muted: #5b6477;
  --color-line: #dde5f2;
  --color-soft: #f5f8ff;
  --color-white: #ffffff;
  --shadow-soft: 0 20px 55px rgba(0, 42, 144, 0.13);
  --radius: 8px;
}

* {
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  color: var(--color-ink);
  background: var(--color-white);
  font-family:
    Inter,
    ui-sans-serif,
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

button,
input,
select,
textarea {
  font: inherit;
}

#app {
  min-height: 100vh;
}
</style>
