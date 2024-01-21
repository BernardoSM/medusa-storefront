<script setup lang="ts">
const client = useMedusaClient();
const { product_categories } = await client.productCategories.list();
const { collections } = await client.collections.list();
</script>

<template>
  <footer class="bg-zinc-50">
    <div class="container py-20 mt-20 grid grid-cols-[repeat(auto-fill,_minmax(280px,_1fr))] gap-4 gap-y-10">
      <div>
        <img src="/logo.svg" height="20" class="h-[20px] w-auto pr-4" />
      </div>
      <div class="text-sm space-y-4">
        <div class="font-medium">
          Categorias
        </div>
        <div v-for="category in product_categories" :key="category.id">
          <nuxt-link :to="`/categorias/${category.id}`" class="hover:underline text-zinc-500 hover:text-zinc-900 transition">
            {{ category.name }}
          </nuxt-link>
        </div>
      </div>
      <div class="text-sm space-y-4">
        <div class="font-medium">
          Coleções
        </div>
        <div v-for="collection in collections" :key="collection.id">
          <nuxt-link :to="`/colecoes/${collection.id}`" class="hover:underline text-zinc-500 hover:text-zinc-900 transition">
            {{ collection.title }}
          </nuxt-link>
        </div>
      </div>
    </div>
  </footer>
</template>