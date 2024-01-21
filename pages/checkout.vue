<script setup lang="ts">
import type { Cart } from '@medusajs/medusa/dist/models/cart'
import type { Order } from '@medusajs/medusa/dist/models/order'
import type { PricedShippingOption } from '@medusajs/medusa/dist/types/pricing'

definePageMeta({
  layout: 'blank'
});

const client = useMedusaClient();
const loading = ref(true);
const buttonLoading = ref(false);
const email = ref();
const shippingAddress = ref({
  company: '',
  first_name: '',
  last_name: '',
  address_1: '',
  address_2: '',
  city: '',
  country_code: 'us', // Como só temos a região dos estados unidos, deixaremos aqui sempre 'us', mas fique a vontade para adicionar outras regiões
  province: '',
  postal_code: '',
  phone: '',
});
const step = ref<'address' | 'shipping' | 'payment' | 'payment-data' | 'complete'>('address');
const shippingOptions = ref<PricedShippingOption[]>();
const selectedShippingId = ref();
const selectedProviderId = ref();
const paymentData = ref({
  credit_card: ''
});

const cart = ref<Cart>();
const order = ref<Order>();

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

async function sendAddress() {
  try {
    buttonLoading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      await client.carts.update(id, { // É obrigatório associar um email ao carrinho
        email: email.value
      })
      await client.carts.update(id, {
        shipping_address: shippingAddress.value
      })
      const response = await client.shippingOptions.listCartOptions(id);
      shippingOptions.value = response.shipping_options;
      step.value = 'shipping';
    }
    
  } finally {
    buttonLoading.value = false;
  }
}

async function sendShippingMethod() {
  try {
    buttonLoading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      await client.carts.addShippingMethod(id, {
        option_id: selectedShippingId.value
      });
      const response = await client.carts.createPaymentSessions(id);
      cart.value = response.cart as unknown as Cart;
      step.value = 'payment';
    }
    
  } finally {
    buttonLoading.value = false;
  }
}

async function sendPaymentSession() {
  try {
    buttonLoading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      const response = await client.carts.setPaymentSession(id, {
        provider_id: selectedProviderId.value
      });
      cart.value = response.cart as unknown as Cart;
      step.value = 'payment-data';
    }
    
  } finally {
    buttonLoading.value = false;
  }
}

async function completeCheckout() {
  try {
    buttonLoading.value = true;

    const id = localStorage.getItem("cart_id")

    if (id) {
      await client.carts.updatePaymentSession(id, selectedProviderId.value, {
        data: paymentData.value,
      });
      const response = await client.carts.complete(id);
      if(response.type === 'order') {
        order.value = response.data;
        step.value = 'complete';
        localStorage.delete("cart_id");
      }
    }
    
  } finally {
    buttonLoading.value = false;
  }
}
</script>

<template>
  <section class="pt-10 bg-zinc-50 min-h-screen">
    <div v-if="loading" class="text-center text-gray-500 pb-20">
      Carregando...
    </div>
    <div v-if="!loading && cart" class="container">
      <div class="grid gap-12 grid-cols-[1fr_520px] max-w-[1120px] mx-auto">
        <div v-if="step === 'complete'" class="text-center pt-10">
          <h2 class="text-2xl font-bold mb-4">
            Pedido feito com sucesso!
          </h2>
          <p>
            Número do pedido: <span class="font-bold">{{ order?.id }}</span> 
          </p>
        </div>
        <div v-else class="h-fit group" :class="step">
          <h2 class="text-xl font-medium mb-4">
            Dados Pessoais
          </h2>
          <div class="space-y-4 hidden group-[.address]:block">
            <div>
              <label for="company" class="text-sm">Email</label>
              <Input id="company" type="company" v-model="email"/>
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label for="first_name" class="text-sm">Nome</label>
                <Input id="first_name" type="first_name" v-model="shippingAddress.first_name"/>
              </div>
              <div>
                <label for="last_name" class="text-sm">Sobrenome</label>
                <Input id="last_name" type="last_name" v-model="shippingAddress.last_name"/>
              </div>
            </div>
            <div>
              <label for="company" class="text-sm">Empresa</label>
              <Input id="company" type="company" v-model="shippingAddress.company"/>
            </div>
            <div>
              <label for="address_1" class="text-sm">Endereço</label>
              <Input id="address_1" type="address_1" v-model="shippingAddress.address_1"/>
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label for="address_2" class="text-sm">Complemento</label>
                <Input id="address_2" type="address_2" v-model="shippingAddress.address_2"/>
              </div>
              <div>
                <label for="city" class="text-sm">Cidade</label>
                <Input id="city" type="city" v-model="shippingAddress.city"/>
              </div>
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label for="province" class="text-sm">Estado</label>
                <Input id="province" type="province" v-model="shippingAddress.province"/>
              </div>
              <div>
                <label for="postal_code" class="text-sm">CEP</label>
                <Input id="postal_code" type="postal_code" v-model="shippingAddress.postal_code"/>
              </div>
            </div>
            <div>
              <label for="phone" class="text-sm">Celular</label>
              <Input id="phone" type="phone" v-model="shippingAddress.phone"/>
            </div>
            <Button class="!mt-8" @click="sendAddress">
              {{ buttonLoading ? 'Carregando...' : 'Continuar' }}
            </Button>
          </div>
          <h2 class="text-xl font-medium mb-4 mt-8">
            Método de entrega
          </h2>
          <div class="space-y-4 hidden group-[.shipping]:block">
            <div class="grid grid-cols-2 gap-4">
              <div v-for="shipping in shippingOptions" :key="shipping.id" 
                class="bg-white rounded-lg border border-zinc-200 text-sm p-5 cursor-pointer hover:bg-zinc-100"
                :class="{'border-zinc-900 bg-zinc-100': selectedShippingId === shipping.id}"
                @click="selectedShippingId = shipping.id"
              >
                <div class="mb-2">
                  {{ shipping.name }}
                </div>
                <div class="text-zinc-500 mb-4">
                  5 - 10 business days
                </div>
                <div class="font-medium">
                  {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(shipping.amount) / 100) }}
                </div>
              </div>
            </div>
            <Button v-if="selectedShippingId" class="!mt-8" @click="sendShippingMethod">
              {{ buttonLoading ? 'Carregando...' : 'Continuar' }}
            </Button>
          </div>
          <h2 class="text-xl font-medium mb-4 mt-8">
            Tipo de Pagamento
          </h2>
          <div class="space-y-4 hidden group-[.payment]:block">
            <div class="grid grid-cols-2 gap-4">
              <div v-for="paymentSession in cart.payment_sessions" :key="paymentSession.id"
                class="bg-white rounded-lg border border-zinc-200 text-sm p-5 cursor-pointer hover:bg-zinc-100 capitalize"
                :class="{'border-zinc-900 bg-zinc-100': selectedProviderId === paymentSession.provider_id}"
                @click="selectedProviderId = paymentSession.provider_id"
              >
                {{ paymentSession.provider_id }}:
                {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(paymentSession.amount) / 100) }}
              </div>
            </div>
            <Button v-if="selectedProviderId" class="!mt-8" @click="sendPaymentSession">
              {{ buttonLoading ? 'Carregando...' : 'Continuar' }}
            </Button>
          </div>
          <h2 class="text-xl font-medium mb-4 mt-8">
            Dados de Pagamento
          </h2>
          <div class="space-y-4 hidden group-[.payment-data]:block">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label for="city" class="text-sm">Número do Cartão (Fake)</label>
                <Input id="city" type="city" v-model="paymentData.credit_card"/>
              </div>
            </div>
            <Button v-if="paymentData.credit_card.length > 0" class="!mt-8" @click="completeCheckout">
              {{ buttonLoading ? 'Carregando...' : 'Continuar' }}
            </Button>
          </div>
        </div>
        <div>
          <h2 class="text-xl font-medium mb-4">
            Resumo do pedido
          </h2>
          <div class="bg-white border border-zinc-200 rounded p-8 h-fit">
            <div class="space-y-2 mb-4">
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
            <div class="flex items-center pt-4 text-lg font-medium">
              <div class="flex-1">
                Total
              </div>
              <div>
                {{ new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(Number(cart.total) / 100) }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>