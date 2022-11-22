<script setup>
import {
  Dialog,
  DialogPanel,
  DialogTitle,
  TransitionChild,
  TransitionRoot,
} from '@headlessui/vue';
import { XMarkIcon } from '@heroicons/vue/24/outline';
import {computed, ref} from 'vue';

const props = defineProps({
  externalCloseButton: Boolean,
  hideClose: Boolean,
  noCloseOnBackdrop: Boolean,
  open: Boolean,
  showBackdrop: Boolean,
  size: {
    type: String,
    default: 'small',
    validator(value) {
      return ['small', 'wide', 'full'].includes(value);
    },
  },
  title: String,
});

const emit = defineEmits(['close']);

const isOpen = ref(props.open);

const sizeClass = computed(() => {
  switch (props.size) {
    case 'wide':
      return 'max-w-2xl';
    case 'full':
      return 'max-w-full';
    default:
      return 'max-w-md';
  }
});

const close = () => {
  isOpen.value = false;

  emit('close', false);
};
const open = () => isOpen.value = true;

defineExpose({
  close,
  open,
});
</script>

<template>
  <TransitionRoot as="template" :show="isOpen">
    <Dialog as="div" class="relative z-10" @close="() => noCloseOnBackdrop ? null : close()">
      <TransitionChild
          as="template"
          enter="ease-in-out duration-500"
          enter-from="opacity-0"
          enter-to="opacity-100"
          leave="ease-in-out duration-500"
          leave-from="opacity-100"
          leave-to="opacity-0"
      >
        <div :class="['fixed inset-0', showBackdrop ? 'bg-gray-500 bg-opacity-75 transition-opacity' : undefined]" />
      </TransitionChild>

      <div class="fixed inset-0 overflow-hidden">
        <div class="absolute inset-0 overflow-hidden">
          <div class="pointer-events-none fixed inset-y-0 right-0 flex max-w-full pl-10 sm:pl-16">
            <TransitionChild
                as="template"
                enter="transform transition ease-in-out duration-500 sm:duration-700"
                enter-from="translate-x-full"
                enter-to="translate-x-0"
                leave="transform transition ease-in-out duration-500 sm:duration-700"
                leave-from="translate-x-0"
                leave-to="translate-x-full"
            >
              <DialogPanel :class="['pointer-events-auto relative w-screen', sizeClass]">
                <TransitionChild
                    v-if="externalCloseButton"
                    as="template"
                    enter="ease-in-out duration-500"
                    enter-from="opacity-0"
                    enter-to="opacity-100"
                    leave="ease-in-out duration-500"
                    leave-from="opacity-100"
                    leave-to="opacity-0"
                >
                  <div class="absolute top-0 left-0 -ml-8 flex pt-4 pr-2 sm:-ml-10 sm:pr-4">
                    <button
                        type="button"
                        class="rounded-md text-gray-300 hover:text-white focus:outline-none focus:ring-2 focus:ring-white"
                        @click="close"
                    >
                      <span class="sr-only">Close panel</span>
                      <XMarkIcon class="h-6 w-6" aria-hidden="true" />
                    </button>
                  </div>
                </TransitionChild>

                <div class="flex h-full flex-col divide-y divide-gray-200 bg-white shadow-xl dark:bg-slate-800 dark:divide-slate-600">
                  <div class="h-0 flex-1 overflow-y-auto">
                    <div class="px-4 py-6 px-4 sm:px-6">
                      <div class="flex items-start justify-between">
                        <DialogTitle v-if="$slots.title || title" class="text-lg font-medium">
                          <slot name="title">{{ title }}</slot>
                        </DialogTitle>

                        <div v-if="!externalCloseButton" class="ml-3 flex h-7 items-center">
                          <button
                              type="button"
                              class="rounded-md bg-white text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2 dark:bg-slate-700 dark:ring-offset-slate-700"
                              @click="close"
                          >
                            <span class="sr-only">Close panel</span>
                            <XMarkIcon class="h-6 w-6" aria-hidden="true" />
                          </button>
                        </div>
                      </div>
                    </div>

                    <div class="relative mt-6 flex-1 px-4 sm:px-6">
                      <slot />
                    </div>
                  </div>

                  <div v-if="$slots.footer" class="flex flex-shrink-0 justify-end  px-4 py-4">
                    <slot name="footer" />
                  </div>
                </div>
              </DialogPanel>
            </TransitionChild>
          </div>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>
