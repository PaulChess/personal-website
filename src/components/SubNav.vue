<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()

const inactiveStyle = 'opacity-20 hover:opacity-50'
</script>

<template>
  <div class="prose m-auto mb-8 select-none">
    <div mb-0 flex="~ gap-3" text-3xl>
      <router-link to="/posts" class="!border-none !font-400" :class="route.path === '/posts' ? '' : inactiveStyle">
        Blog
      </router-link>
    </div>
  </div>
</template>
