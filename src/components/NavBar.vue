<script setup>
import { isDark } from '~/composables'
</script>

<template>
  <header class="header z-40 text-gray-700 dark:text-gray-200">
    <router-link
      w-10 h-10
      absolute
      lg:fixed
      m-6 select-none outline-none
      to="/"
      focusable="false"
    >
      <div inline-block i-ri-chrome-fill w-8 h-8 />
    </router-link>
    <nav class="nav">
      <div class="spacer" />
      <div class="right">
        <router-link to="/posts">
          <span class="lt-md:hidden">Blog</span>
          <div i-ri-book-2-line class="md:hidden" />
        </router-link>
        <router-link to="/talks" class="lt-md:hidden">
          Talks
        </router-link>
        <router-link to="/podcasts" class="lt-md:hidden">
          Podcasts
        </router-link>
        <router-link to="/streams" class="lt-md:hidden">
          Streams
        </router-link>
        <router-link to="/projects">
          <span class="lt-md:hidden">Projects</span>
          <div i-ri-lightbulb-line class="md:hidden" />
        </router-link>
        <router-link to="/sponsors-list" title="Sponsors">
          <div i-ri-heart-line />
        </router-link>
        <router-link to="/bookmarks" title="Bookmarks">
          <div i-ri-bookmark-line />
        </router-link>
        <router-link to="/notes" title="Notes">
          <div i-ri-sticky-note-line />
        </router-link>
        <a href="https://github.com/paulchess" target="_blank" title="GitHub" class="lt-md:hidden">
          <div i-carbon-logo-github />
        </a>
        <toggle-theme />
      </div>
    </nav>
  </header>
</template>

<style scoped>
.header h1 {
  margin-bottom: 0;
}

.nav {
  padding: 2rem;
  width: 100%;
  display: grid;
  grid-template-columns: auto max-content;
  box-sizing: border-box;
}

.nav > * {
  margin: auto;
}

.nav img {
  margin-bottom: 0;
}

.nav a {
  cursor: pointer;
  text-decoration: none;
  color: inherit;
  transition: opacity 0.2s ease;
  opacity: 0.6;
  outline: none;
}

.nav a:hover {
  opacity: 1;
  text-decoration-color: inherit;
}

.nav .right {
  display: grid;
  grid-gap: 1.2rem;
  grid-auto-flow: column;
}

.nav .right > * {
  margin: auto;
}
</style>
