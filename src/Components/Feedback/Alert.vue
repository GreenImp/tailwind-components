<script setup>
import { computed } from 'vue';
import {
  CheckCircleIcon,
  ExclamationTriangleIcon,
  InformationCircleIcon,
  XCircleIcon
} from '@heroicons/vue/20/solid'

const props = defineProps({
  message: String,
  type: String,
});

const styles = {
  default: {},
  danger: {
    container: ['border-red-400', 'bg-red-50'],
    icon: {
      icon: XCircleIcon,
      style: ['text-red-400'],
    },
    text: ['text-red-700'],
  },
  info: {
    container: ['border-blue-400', 'bg-blue-50'],
    icon: {
      icon: InformationCircleIcon,
      style: ['text-blue-400'],
    },
    text: ['text-blue-700'],
  },
  success: {
    container: ['border-green-400', 'bg-green-50'],
    icon: {
      icon: CheckCircleIcon,
      style: ['text-green-400'],
    },
    text: ['text-green-700'],
  },
  warning: {
    container: ['border-yellow-400', 'bg-yellow-50'],
    icon: {
      icon: ExclamationTriangleIcon,
      style: ['text-yellow-400'],
    },
    text: ['text-yellow-700'],
  },
};

const typeStyles = computed(() => styles[props.type] || styles.default);
</script>

<template>
  <div class="border-l-4 p-4" :class="typeStyles.container">
    <div class="flex">
      <div v-if="typeStyles.icon?.icon" class="flex-shrink-0">
        <component :is="typeStyles.icon.icon" class="h-5 w-5" :class="typeStyles.icon?.style" aria-hidden="true" />
      </div>

      <div class="ml-3 text-sm" :class="typeStyles.text">
        <slot>
          <p>{{ message }}</p>
        </slot>
      </div>
    </div>
  </div>
</template>
