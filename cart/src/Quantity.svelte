<script lang="ts">
  import { TrackingEvents } from '@norce/analytics';
  import {
    convertItemToGA4Item,
    createFormatter,
    PlatformAdapters,
    type MountProps,
    type PlatformItem,
  } from '@norce/checkout-lib';
  export let item: PlatformItem<PlatformAdapters.Jetshop>;
  export let api: MountProps['api'];
  export let data: MountProps['data'];
  export let track: MountProps['track'];

  const formatter = createFormatter('sv-SE', data.order.currency);
  let quantity = item.quantity;

  // Reset pending UI when state is idle
  // this must be done since a request could fail
  $: {
    if (api.state === 'idle') {
      quantity = item.quantity;
    }
  }
</script>

<span>{formatter.format(item.price.includingVat)}/pcs</span>
<div class="flex gap-0.5">
  <button
    class="h-6 w-6 bg-[#f7f7f7] disabled:cursor-not-allowed font-mono text-sm"
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
    disabled={quantity <= 1 ? true : false}
    >&minus;
  </button>
  <span
    class="bg-[#f7f7f7] h-6 w-6 flex items-center justify-center text-lg leading-none"
  >
    {quantity}
  </span>
  <button
    class="h-6 w-6 bg-[#f7f7f7] font-mono text-sm"
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
