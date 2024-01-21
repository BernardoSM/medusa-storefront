<script setup lang="ts">
defineProps<{
  image: string;
  title: string;
  variant: string;
  price: number;
  handle: string;
  variantId: string;
}>()

defineEmits(['remove'])
</script>

<template>
  <nuxt-link :to="{path: `/produtos/${handle}`, query: {variant: variantId}}" class="cursor-pointer group block">
    <div class="flex items-center gap-4">
      <div class="aspect-square w-[120px] h-[120px] rounded overflow-hidden">
        <img :src="image" :alt="title" class="w-full h-full object-cover object-center group-hover:scale-105 transition">
      </div>
      <div>
        <h2 class="font-semibold mb-2">
          {{ title }} <span class="font-light">Variante: {{ variant }}</span>
        </h2>
        <div class="mb-2">
          {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(price) }}
        </div>
        <Button size="sm" variant="outline" @click.prevent="$emit('remove')">
          Remover
        </Button>
      </div>
    </div>
  </nuxt-link>
</template>