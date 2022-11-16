<script setup>
import { computed, ref, unref } from 'vue';
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/vue/20/solid';
import PaginateItem from './PaginationItem';

const props = defineProps({
  value: {
    type: Number,
    default: 1,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  limit: {
    type: Number,
    default: 7,
  },
  perPage: {
    type: Number,
    default: 15,
  },
  totalRows: {
    type: Number,
    default: 0,
  },
});

const emit = defineEmits(['change']);

const currentPage = ref(props.value);

const pageCount = computed(() => props.totalRows ? Math.ceil(props.totalRows / props.perPage) : 1);
const rangeStart = computed(() => {
  if (props.perPage) {
    return Math.min((unref(currentPage) - 1) * props.perPage + 1, props.totalRows);
  }

  return 1;
});
const rangeEnd = computed(() => {
  if (props.totalRows) {
    return Math.min(rangeStart.value + props.perPage - 1, props.totalRows);
  }

  return 1;
});
const visiblePages = computed(() => {
  const crnt = unref(currentPage);

  const pages = [crnt];
  let after = crnt;
  let before = crnt;
  let i = 0;

  while((pages.length < props.limit) && (before > 1 || after < pageCount.value)) {
    if ((i % 2 === 0) && after < pageCount.value) {
      pages.push(++after);
    } else if (before > 1) {
      pages.unshift(--before);
    }

    i++;
  }

  // remove the first / last one if we need to show the "..." spacers
  if ((pages.length === props.limit)) {
    if (_.first(pages) > 1) {
      pages.shift();
    }

    if (_.last(pages) < pageCount.value) {
      pages.pop();
    }
  }

  return pages;
});
const showStartSpacer = computed(() => _.first(visiblePages.value) > 1);
const showEndSpacer = computed(() => _.last(visiblePages.value) < pageCount.value);

const setPage = (page) => {
  if (page < 1) {
    page = 1;
  } else if (page > pageCount.value) {
    page = pageCount.value;
  }

  currentPage.value = page;

  emit('change', currentPage.value);
};
</script>

<template>
  <div class="flex items-center justify-between">
    <div class="flex flex-1 justify-between flex-wrap sm:hidden">
      <PaginateItem
          :page="currentPage - 1"
          :disabled="disabled || (currentPage === 1)"
          class="rounded-md"
          @click="setPage(currentPage - 1)"
      >
        <ChevronLeftIcon class="h-5 w-5" aria-hidden="true" />
        <span>Previous</span>
      </PaginateItem>

      <PaginateItem
          :page="currentPage + 1"
          :disabled="disabled || (currentPage === pageCount)"
          class="rounded-md"
          @click="setPage(currentPage + 1)"
      >
        <span>Next</span>
        <ChevronRightIcon class="h-5 w-5" aria-hidden="true" />
      </PaginateItem>
    </div>

    <div
        class="hidden sm:flex sm:flex-1 sm:items-center"
        :class="totalRows ? 'sm:justify-between' : 'sm:justify-end'"
    >
      <div v-if="totalRows">
        <p class="text-sm text-gray-700 dark:text-gray-500">
          Showing
          {{ ' ' }}
          <span class="font-medium">{{ rangeStart }}</span>
          {{ ' ' }}
          to
          {{ ' ' }}
          <span class="font-medium">{{ rangeEnd }}</span>
          {{ ' ' }}
          of
          {{ ' ' }}
          <span class="font-medium">{{ totalRows }}</span>
          {{ ' ' }}
          results
        </p>
      </div>

      <div>
        <nav class="isolate inline-flex -space-x-px rounded-md shadow-sm" aria-label="Pagination">
          <PaginateItem
              :page="currentPage - 1"
              :disabled="disabled || (currentPage === 1)"
              class="rounded-l-md px-2"
              @click="setPage(currentPage - 1)"
          >
            <span class="sr-only">Previous</span>
            <ChevronLeftIcon class="h-5 w-5" aria-hidden="true" />
          </PaginateItem>

          <span
              v-if="showStartSpacer"
              class="relative inline-flex items-center border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700"
          >...</span>

          <template v-if="pageCount > 0">
            <PaginateItem
                v-for="page in visiblePages"
                :page="page"
                :current="currentPage === page"
                :disabled="disabled"
                @click="setPage(page)"
            />

            <span
                v-if="showEndSpacer"
                class="relative inline-flex items-center border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700"
            >...</span>
          </template>

          <PaginateItem
              :page="currentPage + 1"
              :disabled="disabled || (currentPage === pageCount)"
              class="rounded-r-md px-2"
              @click="setPage(currentPage + 1)"
          >
            <span class="sr-only">Next</span>
            <ChevronRightIcon class="h-5 w-5" aria-hidden="true" />
          </PaginateItem>
        </nav>
      </div>
    </div>
  </div>
</template>
