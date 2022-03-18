<script setup>
// This Component is injected as containerComponent in vite.config.js

import { formatDate } from '~/logics'

defineProps({
  frontmatter: {
    type: Object,
    required: true,
  },
})
const route = useRoute()

onMounted(() => {
  if (route.path.includes('/posts')) {
    document.body.classList.remove('font-sans')
    document.body.classList.add('font-mono')
  }
  else {
    document.body.classList.remove('font-mono')
    document.body.classList.add('font-sans')
  }
})
</script>

<template>
  <div v-if="frontmatter.display ?? frontmatter.title" class="prose m-auto mb-8">
    <h1 class="mb-0 text-left">
      {{ frontmatter.display ?? frontmatter.title }}
    </h1>
    <p v-if="frontmatter.date" class="opacity-50 !-mt-2 text-left">
      {{ formatDate(frontmatter.date) }} <span v-if="frontmatter.duration">· {{ frontmatter.duration }}</span>
    </p>
    <p v-if="frontmatter.subtitle" class="opacity-50 !-mt-6 italic">
      {{ frontmatter.subtitle }}
    </p>
  </div>
  <article class="content">
    <slot />
  </article>
  <div v-if="route.path !== '/'" class="prose m-auto mt-8 mb-8 text-left">
    <router-link
      class="font-mono no-underline opacity-50 hover:opacity-75"
      :to="route.path.split('/').slice(0, -1).join('/') || '/'"
    >
      cd ..
    </router-link>
  </div>
</template>
