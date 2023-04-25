<script lang="ts">
  import {
    createFormatter,
    type JetshopDiscountResponse,
    totalShipping,
    calculateTaxAmount,
    totalDiscountedAmount,
  } from '@norce/checkout-lib';
  import type { MountProps } from '@norce/module-adapter-svelte';
  import { onMount } from 'svelte';
  import { culture } from './translations';

  export let api: MountProps['api'];
  export let data: MountProps['data'];
  export let track: MountProps['track'];

  let formEl: HTMLFormElement;
  const formatter = createFormatter('sv-SE', data.order.currency);

  onMount(() => {
    culture.set(data.order.culture);
  });

  const handleAddDiscount = (
    e: Event & {
      readonly submitter: HTMLElement;
    } & {
      currentTarget: EventTarget & HTMLFormElement;
    }
  ) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const code = formData.get('discountCode') as string;
    api.applyDiscount(code);
  };

  $: formError =
    api.response?.error?.__type === 'FormError'
      ? (api.response.error as Extract<
          JetshopDiscountResponse['error'],
          { __type: 'FormError' }
        >)
      : undefined;
  $: {
    const successToast =
      api.response?.error?.__type === 'ToastError' &&
      api.response?.error?.variant === 'success'
        ? api.response.error
        : undefined;
    if (successToast) {
      formEl.reset();
    }
  }
  $: pulsingClass =
    api.state === 'pending'
      ? 'text-transparent bg-black/20 rounded animate-pulse'
      : '';

  let showDiscount = false;
</script>

<div class="grid gap-2 mb-5">
  <!-- Discount -->
  <div class="bg-white p-5 grid gap-2">
    {#if !showDiscount}
      <button
        class="flex underline underline-offset-4"
        on:click={() => (showDiscount = true)}
      >
        Enter discount code
      </button>
    {:else}
      <form bind:this={formEl} on:submit={handleAddDiscount} class="contents">
        <!-- They really wanted this even though it's bad -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <label
          class="block underline underline-offset-4"
          for="discountCode"
          on:click={() => (showDiscount = false)}
        >
          Enter discount code
        </label>
        <div class="flex md:w-64">
          <input
            id="discountCode"
            type="text"
            name="discountCode"
            placeholder="Enter your code here and click OK"
            class="flex-1 py-1 border border-r-0 border-zinc-400 outline-none pl-2 placeholder:opacity-50"
            required
          />
          <button
            type="submit"
            class="inline-flex items-center justify-center bg-zinc-500 text-white px-2 hover:bg-zinc-600 focus-visible:bg-zinc-600"
          >
            OK
          </button>
        </div>

        {#if formError?.fields?.discountCode}
          <span
            class="justify-self-start bg-red-200 border border-red-600 px-2 py-1 my-1"
          >
            {formError.fields.discountCode}
          </span>
        {/if}
      </form>
    {/if}

    {#if data.order?.cart?.discounts && data.order?.cart?.discounts.length > 0}
      <div class="grid gap-1 text-lg leading-none my-2">
        <h3 class="font-bold">Active discounts</h3>
        {#each data.order?.cart?.discounts || [] as discount}
          <div class="flex items-center justify-between group">
            <div class="flex items-center">
              <span class="flex items-center gap-1">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke-width="1.5"
                  stroke="currentColor"
                  class="w-5 h-5 inline text-green-700"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M4.5 12.75l6 6 9-13.5"
                  />
                </svg>

                {discount.name}
              </span>
              {#if discount.code}
                <button
                  class="ml-2 px-1 text-zinc-300 md:hidden group-hover:block hover:text-red-700 font-mono leading-none"
                  on:click={() => api.removeDiscount(discount.code)}
                >
                  &times;
                </button>
              {/if}
            </div>
            <span class={pulsingClass}>
              - {formatter.format(discount.value.includingVat)}
            </span>
          </div>
        {/each}
      </div>
      <div class="flex font-bold text-lg leading-none">
        <span class="flex-1">Total discount</span>
        <span class={pulsingClass}>
          {formatter.format(totalDiscountedAmount(data.order?.cart?.discounts))}
        </span>
      </div>
    {/if}
  </div>

  <!-- Summary -->
  <div class="bg-white p-5 text-lg leading-none grid gap-1.5">
    <div class="flex items-center justify-between font-bold">
      <span>Total items:</span>
      <span class={pulsingClass}>
        {formatter.format(data.order.cart.total.includingVat)}
      </span>
    </div>

    <div class="flex items-center justify-between">
      <span>Freight charge:</span>
      <span class={pulsingClass}>
        {formatter.format(totalShipping([...(data.order.shippings || [])]))}
      </span>
    </div>

    <div
      class="flex items-center justify-between font-bold text-2xl leading-none mt-1"
    >
      <span>Total incl. VAT</span>
      <span class={pulsingClass}>
        {formatter.format(data.order.total.includingVat)}
      </span>
    </div>

    <div class="flex items-center justify-between">
      <span>25% VAT</span>
      <span class={pulsingClass}>
        {formatter.format(calculateTaxAmount(data.order.total))}
      </span>
    </div>
  </div>
</div>

<style global lang="postcss">
  @import 'tailwindcss/base';
  @import 'tailwindcss/components';
  @import 'tailwindcss/utilities';
</style>
