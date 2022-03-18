<template>
  <ul>
    <template v-if="!posts.length">
      <div py2 op50>
        { nothing here yet }
      </div>
    </template>
    <router-link
      v-for="route in posts" :key="route.path"
      class="item block font-normal mb-6 mt-2 no-underline"
      :to="route.path"
    >
      <li class="no-underline list-none">
        <div class="title text-lg" flex items-center>
          <div class="i-ri-article-fill" mr-1 />
          {{ route.title }}
          <!-- <sup
            v-if="route.lang === 'zh'"
            class="text-xs border border-current rounded px-1 pb-0.2"
          >中文</sup> -->
        </div>
        <div class="time opacity-70 text-sm mt-1">
          {{ formatDate(route.date) }}
          <span v-if="route.duration" class="opacity-80 ml-1">· {{ route.duration }}</span>
        </div>
      </li>
    </router-link>
  </ul>
</template>

<script setup lang="ts">
import { formatDate } from '~/logics'

export interface Post {
  path: string
  title: string
  date: string
  lang?: string
  duration?: string
}

const props = defineProps<{
  type?: string
  posts?: Post[]
}>()

const router = useRouter()
const routes: Post[] = router.getRoutes()
  .filter(i => i.path.startsWith('/posts') && i.meta.frontmatter?.date)
  .sort((a, b) => +new Date(b.meta.frontmatter?.date) - +new Date(a.meta.frontmatter?.date))
  .map(i => ({
    path: i.path,
    title: i.meta.frontmatter.title,
    date: i.meta.frontmatter.date,
    lang: i.meta.frontmatter.lang,
    duration: i.meta.frontmatter.duration,
  }))

const posts = computed(() => props.posts || routes)
</script>

<style lang="less" scoped>

</style>
