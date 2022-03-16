<script setup lang="ts">
const currentYear = ref(new Date().getFullYear())
</script>

<template>
  <div class="mt-10 mb-6 prose m-auto opacity-50 flex">
    <span class="text-sm">Powered By <a target="_blank" href="https://github.com/antfu/antfu.me/" style="color:inherit">antfu.me</a> {{ currentYear }}-PRESENT © PaulChess</span>
    <div class="flex-auto" />
  </div>
</template>
