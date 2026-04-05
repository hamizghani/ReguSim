<template>
  <router-view />
</template>

<script setup>
// Use Vue Router to manage pages
</script>

<style>
/* ======== REGUSIM DESIGN SYSTEM ======== */
:root {
  /* Background */
  --bg-base: #FFFFFF;
  --bg-surface: #FFFFFF;
  --bg-warm: #FAF9F6;
  --bg-card: #FFFFFF;
  --bg-card-hover: #FEFDFB;
  --bg-elevated: #F5F3EF;
  --bg-glass: rgba(255, 255, 255, 0.9);

  /* Primary — deep navy */
  --accent: #1E3A5F;
  --accent-light: #2B4F7A;
  --accent-dark: #152B47;
  --accent-bg: rgba(30, 58, 95, 0.07);
  --accent-border: rgba(30, 58, 95, 0.22);

  /* Secondary — teal */
  --secondary: #2F7F7A;
  --secondary-light: #3A9994;
  --secondary-dark: #236660;
  --secondary-bg: rgba(47, 127, 122, 0.08);

  /* Semantic */
  --amber: #D4A843;
  --amber-light: #E0BE6A;
  --emerald: #2F7F7A;
  --rose: #C4505A;
  --blue: #1E3A5F;

  /* Text */
  --text-primary: #1A1A1A;
  --text-secondary: #5A5A5A;
  --text-muted: #9A9A9A;
  --text-inverse: #FFFFFF;

  /* Border */
  --border: #E8E5E0;
  --border-light: #F0EDE8;
  --border-accent: rgba(30, 58, 95, 0.25);

  /* Radius — small, refined */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 10px;
  --radius-xl: 12px;

  /* Shadows — subtle, warm */
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.02);
  --shadow-md: 0 2px 8px rgba(0, 0, 0, 0.06), 0 1px 3px rgba(0, 0, 0, 0.04);
  --shadow-lg: 0 4px 16px rgba(0, 0, 0, 0.08), 0 2px 6px rgba(0, 0, 0, 0.04);

  /* Fonts */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-serif: 'Playfair Display', Georgia, 'Times New Roman', serif;
  --font-mono: 'Fira Code', 'Consolas', monospace;
}

/* Global reset */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

body {
  background: var(--bg-base);
  color: var(--text-primary);
  font-family: var(--font-sans);
}

#app {
  font-family: var(--font-sans);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--text-primary);
  background: var(--bg-base);
  min-height: 100vh;
}

/* Scrollbar */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
::-webkit-scrollbar-track {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: #D5D0C8;
  border-radius: 10px;
}
::-webkit-scrollbar-thumb:hover {
  background: #B8B0A4;
}

button {
  font-family: inherit;
}

a {
  color: var(--accent-dark);
  text-decoration: none;
}
a:hover {
  color: var(--accent);
}

.mono {
  font-family: var(--font-mono);
}
</style>
