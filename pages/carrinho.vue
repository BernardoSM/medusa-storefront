<script setup lang="ts">
import type { Cart } from '@medusajs/medusa/dist/models/cart'

const client = useMedusaClient();
const loading = ref(true);

const cart = ref<Cart>();

onMounted(async() => {
  try {
    loading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      const response = await client.carts.retrieve(id);
      console.log(response);
      cart.value = response.cart as unknown as Cart;
    }
  } finally {
    loading.value = false;
  }
})

async function removeItemFromCart(lineItemId: string) {
  try {
    loading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      const response = await client.carts.lineItems.delete(id, lineItemId);
      cart.value = response.cart as unknown as Cart;
    }
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <section class="container pt-24">
    <div v-if="loading" class="text-center text-gray-500 pb-20">
      Carregando...
    </div>
    <div v-if="!loading && cart">
      <h1 class="text-4xl font-extrabold mb-10">
        Carrinho
      </h1>
      <div class="grid gap-10 grid-cols-[1fr_440px]">
        <div class="space-y-4">
          <CartProductCard 
          v-for="lineItem in cart.items"
            :key="lineItem.id"
            :image="(lineItem.variant.product.thumbnail as string)" 
            :title="lineItem.title" 
            :variant="lineItem.variant.title" 
            :variant-id="lineItem.variant.id" 
            :price="lineItem.unit_price / 100" 
            :handle="(lineItem.variant.product.handle as string)" 
            @remove="removeItemFromCart(lineItem.id)" 
          />
        </div>
        <div class="bg-zinc-50 rounded p-8 h-fit">
          <h2 class="text-xl font-medium mb-2">
            Resumo do pedido
          </h2>
          <div class="flex items-center text-sm border-b border-zinc-200 py-4">
            <div class="flex-1 text-gray-500">
              Subtotal
            </div>
            <div>
              {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(cart.subtotal) / 100) }}
            </div>
          </div>
          <div class="flex items-center text-sm border-b border-zinc-200 py-4">
            <div class="flex-1 text-gray-500">
              Desconto
            </div>
            <div>
              - {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(cart.discount_total) / 100) }}
            </div>
          </div>
          <div class="flex items-center py-4 text-lg font-medium mb-4">
            <div class="flex-1">
              Total
            </div>
            <div>
              {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(cart.total) / 100) }}
            </div>
          </div>
          <nuxt-link to="/checkout">
            <Button size="lg" class="w-full">
              Fazer pedido
            </Button>
          </nuxt-link>
        </div>
      </div>
    </div>
  </section>
</template>