<script setup>
import {
  UserCircleIcon,
} from '@heroicons/vue/24/outline';

defineProps({
  alt: String,
  src: String,
});
</script>

<template>
  <img
      v-if="src"
      class="rounded-full"
      :src="src"
      :alt="alt"
  />
  <UserCircleIcon v-else class="rounded-full" aria-hidden="true" />
</template>
