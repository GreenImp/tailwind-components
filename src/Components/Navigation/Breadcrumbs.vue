<script setup>
import { ChevronRightIcon, HomeIcon } from '@heroicons/vue/20/solid';

const props = defineProps({
  pages: {
    type: Array,
    required: true,
  },
  tag: {
    default: 'a',
  },
});
</script>

<template>
  <nav class="flex py-2 px-4 sm:px-6 md:px-8 border-b border-gray-200 bg-gray-100" aria-label="Breadcrumb">
    <ol role="list" class="flex flex-wrap items-center gap-x-4 gap-y-4">
      <li>
        <div>
          <component :is="tag" :href="route('hub.dashboard')" class="text-gray-400 hover:text-gray-500">
            <HomeIcon class="h-5 w-5 flex-shrink-0" aria-hidden="true" />
            <span class="sr-only">Home</span>
          </component>
        </div>
      </li>

      <li v-for="page in pages" :key="page.name">
        <div class="flex items-center">
          <ChevronRightIcon class="h-5 w-5 flex-shrink-0 text-gray-400" aria-hidden="true" />

          <component
              :is="page.href ? tag : 'span'"
              :href="page.href"
              class="ml-4 text-sm font-medium text-gray-500"
              :class="page.href ? 'hover:text-gray-700' : undefined"
              :aria-current="page.current ? 'page' : undefined"
          >{{ page.name }}</component>
        </div>
      </li>
    </ol>
  </nav>
</template>
