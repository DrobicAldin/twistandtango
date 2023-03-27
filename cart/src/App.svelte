<script lang="ts">
  import type { MountProps } from '@norce/module-adapter-svelte';
  import CartItem from './CartItem.svelte';
  import { onMount } from 'svelte';
  import { culture } from './translations';
  import Logo from './Logo.svelte';

  export let api: MountProps['api'];
  export let data: MountProps['data'];
  export let track: MountProps['track'];

  onMount(() => {
    culture.set(data.order.culture);
  });
</script>

<Logo className="mx-auto mt-4 mb-2" />
<div class="grid gap-2 text-lg mb-10">
  <h2 class="uppercase text-center my-2 text-[1.3rem]">Review your order</h2>
  <ul class="grid gap-2">
    {#each data.order.cart.items as item}
      <CartItem {item} {api} {data} {track} />
    {/each}
  </ul>
</div>

<style global lang="postcss">
  @import 'tailwindcss/base';
  @import 'tailwindcss/components';
  @import 'tailwindcss/utilities';
</style>
