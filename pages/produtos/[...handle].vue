<script setup lang="ts">
const { params, query } = useRoute();
const router = useRouter();
const client = useMedusaClient();
const { products } = await client.products.list({handle: params.handle[0] as string});

const selectedVariantId = ref(query.variant);
const loading = ref(false);

const product = computed(() => {
  return products[0];
})

const firstImage = computed(() => {
  if(!product.value.images) return null;

  return product.value.images[0];
})

const otherImages = computed(() => {
  if(!product.value.images) return [];
  if(product.value.images.length <= 1) return [];

  product.value.images.shift();

  return product.value.images;
})

const currentPrice = computed(() => {
  if(!selectedVariantId.value) return 0;

  const currentVariant = product.value.variants.find(e => e.id === selectedVariantId.value);
  if(!currentVariant) return 0;

  return currentVariant.prices[0].amount / 100;
})

async function addToCart() {
  try {
    loading.value = true;

    let cartId = localStorage.getItem("cart_id");

    if(!cartId) {
      const response = await client.carts.create();
      cartId = response.cart.id;
      localStorage.setItem("cart_id", response.cart.id);
    }

    await client.carts.lineItems.create(cartId, {
      variant_id: selectedVariantId.value as string,
      quantity: 1
    })

    router.push('/carrinho');
    
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <section class="container grid gap-10 grid-cols-2 pt-24">
    <div>
      <div class="aspect-square w-full h-auto overflow-hidden rounded-lg mb-4">
        <img v-if="firstImage" :src="firstImage.url" :alt="product.title" />
      </div>
      <div class="grid grid-cols-[repeat(auto-fill,_minmax(200px,_1fr))] gap-4">
        <div class="aspect-square rounded-lg overflow-hidden" v-for="image in otherImages" :key="image.id">
          <img :src="image.url" :alt="product.title" class="w-full h-full object-cover object-center">
        </div>
      </div>
    </div>
    <div>
      <h1 class="text-2xl font-extrabold mb-2">
        {{ product.title }}
      </h1>
      <div class="font-light text-3xl mb-4">
        {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(currentPrice) }}
      </div>
      <p class="text-base text-zinc-500 mb-8">
        {{ product.description }}
      </p>
      <div class="mb-12" v-if="product.variants">
        <div class="text-sm mb-2">Variantes</div>
        <div class="grid grid-cols-[repeat(auto-fill,_minmax(90px,_1fr))] gap-3">
          <ProductVariant 
            v-for="variant in product.variants" 
            :key="variant.id" 
            :variant="(variant.title as string)" 
            :active="selectedVariantId === variant.id" 
            @click="selectedVariantId = (variant.id as string)"
          />
        </div>
      </div>
      <Button variant="default" size="lg" @click="addToCart()">
        {{ loading ? 'Carregando...' : 'Adicionar ao carrinho' }}
      </Button>
    </div>
  </section>
</template>