<script setup lang="ts">
const client = useMedusaClient();
const { products } = await client.products.list();
</script>

<template>
  <div>
    {{ products }}
  </div>
</template>