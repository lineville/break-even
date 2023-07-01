<script lang="ts">
  import { createEventDispatcher } from "svelte";

  import { fly } from "svelte/transition";
  export let cards = [];
  export let visible = true;
  const dispatch = createEventDispatcher();
</script>

<div style="display: inline-block;">
  {#each cards as card, idx (card + idx)}
    <li
      in:fly={{ x: (idx === 0 ? -1 : 1) * 2000, duration: 500, delay: 300 }}
      out:fly={{ x: -2000, duration: 800 }}
      on:outroend={() => dispatch("gone", { card: card })}
    >
      <div class="card">
        <div class="card-image">
          <figure class="image is-128x128">
            {#if visible || idx === 0}
              <img src={`./images/${card}.jpg`} alt="playing card" />
            {:else}
              <img src="./images/Gray_back.jpg" alt="playing card" />
            {/if}
          </figure>
        </div>
      </div>
    </li>
  {/each}
</div>

<style>
  li {
    display: inline-block;
    margin: 5px 5px 30px 5px;
  }
</style>
