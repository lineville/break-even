<script lang="ts">
  import { createEventDispatcher } from "svelte";

  import { fly } from "svelte/transition";
  export let cards = [];
  export let visible = true;
  export let isSplit = false;
  export let isDealer = false;
  const dispatch = createEventDispatcher();

  let innerWidth = 0;
  $: isTouch = innerWidth <= 768;

  // TODO fix transitions and spacing as split cards are added
</script>

<svelte:window bind:innerWidth />

<div style="display: inline-block;">
  {#each cards as card, idx (card + idx)}
    <li
      in:fly={{ y: -2000, duration: 500 }}
      out:fly={{ x: -2000, duration: 500 }}
      on:outroend={() => dispatch("gone", { card: card })}
      class:inline={!isSplit || !isTouch || isDealer}
    >
      <div class="card">
        <div class="card-image">
          <figure
            class={`image ${
              isTouch ? (isSplit ? "is-64x64" : "is-96x96") : "is-128x128"
            }`}
          >
            {#if visible || idx === 0}
              <img src={`./images/${card}.jpg`} alt={`playing card ${card}`} />
            {:else}
              <img src="./images/Gray_back.jpg" alt={`playing card ${card}`} />
            {/if}
          </figure>
        </div>
      </div>
    </li>
  {/each}
</div>

<style>
  li {
    list-style-type: none;
    margin: 5px 5px 30px 5px;
  }
  @media screen and (max-width: 768px) {
    li:nth-of-type(n + 4) {
      margin-top: -80px;
    }
  }

  .inline {
    display: inline-block;
  }
  @media screen and (max-width: 768px) {
    li:not(.inline):nth-of-type(n + 2) {
      margin-top: -60px;
    }
  }
</style>
