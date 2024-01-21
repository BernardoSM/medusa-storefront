<template>
  <main>
    <div class="flex items-center justify-center py-3 px-4 fixed top-0 left-0 w-full pointer-events-none z-10">
      <div class="rounded-full py-2 px-6 bg-white border border-zinc-200 shadow-sm flex items-center gap-2 pointer-events-auto">
        <img src="/logo.svg" height="20" class="h-[20px] w-auto pr-4" />
        <nuxt-link to="/" active-class="[&>*]:font-bold">
          <Button variant="ghost" size="sm">
            Início
          </Button>
        </nuxt-link>
        <nuxt-link to="/categorias" active-class="[&>*]:font-bold">
          <Button variant="ghost" size="sm">
            Categorias
          </Button>
        </nuxt-link>
        <nuxt-link to="/carrinho" active-class="[&>*]:font-bold">
          <Button variant="ghost" size="sm">
            Carrinho
          </Button>
        </nuxt-link>
      </div>
    </div>
    <slot />
    <Footer />
  </main>
</template>