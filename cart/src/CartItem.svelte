<script lang="ts">
  import type { MountProps } from '@norce/module-adapter-svelte';
  import { createFormatter, convertItemToGA4Item } from '@norce/checkout-lib';
  import { t } from './translations';
  import { TrackingEvents } from '@norce/analytics';
  export let item: MountProps['data']['order']['cart']['items'][number];
  export let api: MountProps['api'];
  export let data: MountProps['data'];
  export let track: MountProps['track'];

  const formatter = createFormatter(data.order.culture, data.order.currency);

  $: pulsingClass =
    api.state === 'pending'
      ? 'text-transparent bg-black/20 rounded animate-pulse'
      : '';

  let quantity = item.quantity;
  let total = item.total.includingVat;

  // Reset pending UI when state is idle
  // this must be done since a request could fail
  $: {
    if (api.state === 'idle') {
      quantity = item.quantity;
      total = item.total.includingVat;
    }
  }

  const { id, ...variantData } = item.attributes?.productVariant || {};
  const variant = Object.values(variantData).join(' - ');

  let imageError = false;
</script>

<li class="flex gap-4 p-2 h-36 bg-white text-xs">
  {#if item.imageUrl && !imageError}
    <img
      src={item.imageUrl}
      alt={imageError ? 'No image available' : item.name}
      class="object-contain aspect-[2/3] h-full bg-[#f7f7f7]"
      on:error={() => {
        imageError = true;
      }}
    />
  {:else}
    <div
      class="aspect-[2/3] h-full flex items-center text-center bg-[#f7f7f7] text-sm capitalize"
    >
      Image not found
    </div>
  {/if}

  <!-- Item details -->
  <div class="grid grid-cols-3 w-full">
    <div class="grid gap-2 content-center">
      <a href={item.url} class="hover:underline underline-offset-4">
        {#if variant}
          <h2>{item.name} - {variant}</h2>
        {:else}
          <h2>{item.name}</h2>
        {/if}
      </a>

      <span class="text-xs">Article number: {item.sku}</span>
    </div>

    <!-- Price / Quantity -->
    <div class="flex items-center gap-2">
      <span>{formatter.format(item.price.includingVat)}/pcs</span>
      <button
        class="h-6 w-6 bg-[#f7f7f7]"
        on:click={() => {
          quantity = quantity - 1;
          api.updateItem({
            itemReference: item.reference,
            quantity: String(quantity),
          });
          track(TrackingEvents.RemoveFromCart, {
            items: [convertItemToGA4Item({ ...item, quantity: quantity })],
            currency: data.order.currency,
            value: item.price?.includingVat,
          });
        }}
        >&minus;
      </button>
      <span>{quantity}</span>
      <button
        class="h-6 w-6 bg-[#f7f7f7]"
        on:click={() => {
          quantity = quantity + 1;
          api.updateItem({
            itemReference: item.reference,
            quantity: String(quantity),
          });
          track(TrackingEvents.AddToCart, {
            items: [convertItemToGA4Item({ ...item, quantity: quantity })],
            currency: data.order.currency,
            value: item.price?.includingVat,
          });
        }}
        >&plus;
      </button>
    </div>

    <!-- Total -->
    <div class="flex items-center gap-2">
      <div class="grid gap-2 flex-1 text-end">
        <span class="text-sm">{formatter.format(total)}</span>
        {#each item.discounts as discount}
          <span class="text-green-800">
            {discount.name}: -{formatter.format(discount.value.includingVat)}
          </span>
        {/each}
      </div>
      <button
        class="h-5 w-5 text-[#f7f7f7] hover:text-red-700 text-lg leading-none"
      >
        &times;
      </button>
    </div>
  </div>
</li>
