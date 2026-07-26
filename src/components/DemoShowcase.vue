<template>
  <NomadsShowcase @apply-template="(payload) => $emit('apply-template', payload)" />
</template>

<script setup lang="ts">
import NomadsShowcase from './NomadsShowcase.vue';

defineEmits<{
  (e: 'apply-template', payload: any): void;
}>();
</script>
