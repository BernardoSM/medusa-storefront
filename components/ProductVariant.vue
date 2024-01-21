<script setup lang="ts">
defineProps<{
  variant: string;
  active?: boolean;
}>()
</script>

<template>
  <button class="rounded bg-white border border-zinc-300 p-3 text-sm flex items-center justify-center hover:bg-zinc-100 transition"
  :class="{'!bg-zinc-900 !text-white !border-zinc-900': active}">
    {{ variant }}
  </button>
</template>