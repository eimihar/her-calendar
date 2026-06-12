<script setup lang="ts">
import Icon from './Icon.vue'
import { computed } from 'vue'

const props = defineProps<{
  text?: string
  icon?: string
  variant?: 'primary' | 'secondary' | 'ghost'
}>()

const isIconOnly = computed(() => props.icon && !props.text)
</script>

<template>
  <button
    class="inline-flex items-center justify-center rounded-xl font-medium transition-all shadow-lg disabled:opacity-50 disabled:cursor-not-allowed h-10"
    :class="{
      'px-4 gap-2': !isIconOnly,
      'px-2': isIconOnly,
      'bg-amber-500 text-white hover:bg-amber-600': variant === 'primary' || !variant,
      'bg-gray-100 text-gray-600 hover:bg-gray-200': variant === 'secondary',
      'bg-transparent text-gray-600 hover:bg-gray-100': variant === 'ghost'
    }"
  >
    <Icon v-if="icon" :icon="icon" class="w-4 h-4" />
    <span v-if="text">{{ text }}</span>
  </button>
</template>
