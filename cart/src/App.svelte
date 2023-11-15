<script lang="ts">
  import type { MountProps } from "@norce/module-adapter-svelte";
  import CartItem from "./CartItem.svelte";
  import { onMount } from "svelte";
  import { culture } from "./translations";
  import Logo from "./Logo.svelte";
  import type { PlatformItem, PlatformAdapters } from "@norce/checkout-lib";

  export let api: MountProps["api"];
  export let data: MountProps["data"];
  export let track: MountProps["track"];

  $: items = (data.order?.cart?.items || []) as unknown as
    | PlatformItem<typeof PlatformAdapters.Jetshop>[]
    | undefined;

  onMount(() => {
    culture.set(data.order.culture);
  });
</script>

<a class="block mx-auto mt-4 mb-2 w-fit" href="/">
  <Logo />
</a>
<div class="grid gap-2 text-lg">
  <h2 class="uppercase text-center my-2 text-[1.3rem]">Review your order</h2>
  <ul class="grid gap-2">
    {#each items as item}
      <CartItem {item} {api} {data} {track} />
    {/each}
  </ul>
</div>

<style global lang="postcss">
  @import "tailwindcss/base";
  @import "tailwindcss/components";
  @import "tailwindcss/utilities";
</style>
