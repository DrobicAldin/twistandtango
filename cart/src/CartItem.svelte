<script lang="ts">
  import type { MountProps } from '@norce/module-adapter-svelte';
  import { createFormatter, convertItemToGA4Item } from '@norce/checkout-lib';
  import { t } from './translations';
  import { TrackingEvents } from '@norce/analytics';
  import Quantity from './Quantity.svelte';
  export let item: MountProps['data']['order']['cart']['items'][number];
  export let api: MountProps['api'];
  export let data: MountProps['data'];
  export let track: MountProps['track'];

  const formatter = createFormatter('sv-SE', data.order.currency);

  $: pulsingClass =
    api.state === 'pending'
      ? 'text-transparent bg-black/20 rounded animate-pulse'
      : '';

  let total = item.total.includingVat;

  // Reset pending UI when state is idle
  // this must be done since a request could fail
  $: {
    if (api.state === 'idle') {
      total = item.total.includingVat;
    }
  }

  const { id, ...variantData } = item.attributes?.productVariant || {};
  const variant = Object.values(variantData).join(' - ');

  let imageError = false;
</script>

<li class="flex gap-4 p-2 bg-white leading-none">
  <div class="aspect-[2/3] w-1/4 md:h-36 md:w-auto">
    {#if item.imageUrl && !imageError}
      <a href={item.imageUrl} target="_blank">
        <img
          src={item.imageUrl.replace('original', 'thumbs')}
          alt={item.name}
          class="object-contain bg-[#f7f7f7]"
          on:error={() => {
            imageError = true;
          }}
        /></a
      >
    {:else}
      <div
        class="flex items-center text-center justify-center text-lg capitalize bg-[#f7f7f7] w-full aspect-[2/3]"
      >
        Image <br />not found
      </div>
    {/if}
  </div>

  <!-- Item details -->
  <div class="grid grid-cols-3 w-full">
    <div
      class="grid gap-1 max-h-36 md:pt-1 content-start col-span-2 md:col-span-1"
    >
      <a href={item.url} class="hover:underline underline-offset-4">
        {#if variant}
          <h2>{item.name} - {variant}</h2>
        {:else}
          <h2>{item.name}</h2>
        {/if}
      </a>

      <span class="text-xs hidden md:block">Article number: {item.sku}</span>

      <div
        class="self-start gap-1 max-h-36 flex flex-col md:hidden leading-none"
      >
        <Quantity {item} {api} {track} {data} />
      </div>
    </div>

    <!-- Price / Quantity -->
    <div
      class="items-center justify-center self-start gap-2 max-h-36 hidden md:flex pt-3 leading-none"
    >
      <Quantity {item} {api} {track} {data} />
    </div>

    <!-- Total -->
    <div class="flex flex-col md:flex-row items-end md:items-start gap-2">
      <button
        class="h-5 w-5 text-zinc-300 hover:text-red-700 text-base leading-none md:self-center md:order-2 font-mono"
      >
        &times;
      </button>
      <div
        class="grid gap-2 flex-1 text-end justify-end md:order-1 md:content-start content-end"
      >
        <span class={`${pulsingClass}`}>{formatter.format(total)}</span>
        {#each item.discounts as discount}
          <span class="text-green-800 hidden md:block">
            {discount.name}:
            <span class={`${pulsingClass}`}>
              {' '}-{formatter.format(discount.value.includingVat)}
            </span>
          </span>
        {/each}
      </div>
    </div>

    <div class="md:hidden text-end content-end col-span-full">
      {#each item.discounts as discount}
        <span class="text-green-800">
          {discount.name}:
          <span class={`${pulsingClass}`}>
            {' '}-{formatter.format(discount.value.includingVat)}
          </span>
        </span>
      {/each}
    </div>
  </div>
</li>
