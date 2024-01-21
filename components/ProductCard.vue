<script setup lang="ts">
defineProps<{
  image: string;
  title: string;
  price: number;
  handle: string;
  variantId: string;
}>()
</script>

<template>
  <nuxt-link :to="{path: `/produtos/${handle}`, query: {variant: variantId}}" class="cursor-pointer group">
    <div class="w-full h-auto aspect-[16/12] overflow-hidden rounded-lg">
      <img :src="image" :alt="title" class="w-full h-full object-center object-cover group-hover:scale-105 transition">
    </div>
    <div class="py-4 px-4">
      <h3 class="mb-1">
        {{ title }}
      </h3>
      <div class="font-bold">
        {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(price) }}
      </div>
    </div>
  </nuxt-link>
</template>