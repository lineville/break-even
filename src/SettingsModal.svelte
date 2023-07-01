<script lang="ts">
  export let toggleShowHandTotal;
  export let showHandTotal;
  export let newGameState;
  export let hintEnabled;
</script>


<div id="settings-modal" class="modal">
  <div class="modal-background" />
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Settings</p>
      <button class="delete" aria-label="close" />
    </header>
    <section class="modal-card-body">
      <button class="button is-info is-outlined" on:click={toggleShowHandTotal}>
        {#if showHandTotal}
          <span>Hide Total</span>
          <span class="icon is-small">
            <i class="fas fa-eye-slash" />
          </span>
        {:else}
          <span>Show Total</span>
          <span class="icon is-small">
            <i class="fas fa-eye" />
          </span>
        {/if}
      </button>

      <button class="button is-danger is-outlined" on:click={newGameState}>
        <span>New Game</span>
        <span class="icon is-small">
          <i class="fas fa-rotate" />
        </span>
      </button>

      <button
        class="button is-primary is-outlined"
        on:click={() => {
          hintEnabled = !hintEnabled;
        }}
      >
        <span>{hintEnabled ? "Hide Hint" : "Ask Don"}</span>
        <span class="icon is-medium">
          <i class="fas fa-question-circle" />
        </span>
      </button>

      <a
        class="button is-dark is-outlined"
        href="https://github.com/lineville/break-even"
        target="_blank"
        id="github-link"
      >
        <span>Source Code</span>
        <span class="icon is-large">
          <i class="fab fa-github" />
        </span>
      </a>
    </section>

    <footer class="modal-card-foot" />
  </div>
</div>