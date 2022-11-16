<script setup>
const props = defineProps({
  current: {
    type: Boolean,
    default: false,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  href: {
    type: String,
    default: '#',
  },
  label: String,
  page: {
    type: Number,
    required: true,
  },
});

const emit = defineEmits(['click']);

const onClick = () => {
  if (props.disabled || props.current) {
    return;
  }

  emit('click', props.page);
};
</script>

<template>
  <component
      :is="(current || disabled) ? 'span' : 'a'"
      :href="disabled ? '#' : href"
      class="relative inline-flex items-center border px-4 py-2 text-sm font-medium focus:z-20"
      :class="[
          current ? 'z-10 border-indigo-500 bg-indigo-50 text-indigo-600' : 'border-gray-300 bg-white text-gray-500 hover:bg-gray-50',
          disabled ? 'text-gray-300 cursor-not-allowed' : undefined,
      ]"
      :disabled="disabled"
      @click.prevent="onClick"
  >
    <slot>{{ label || page }}</slot>
  </component>
</template>
