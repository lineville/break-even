<script lang="ts">
  export let newGameState;
</script>

<div id="confirm-new-game-modal" class="modal">
  <div class="modal-background" />
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Reset your balance?</p>
      <button class="delete" aria-label="close" />
    </header>
    <section class="modal-card-body">

      <button class="button is-primary" on:click={newGameState}>
        <span>Yes</span>
        <span class="icon is-small">
          <i class="fas fa-check" />
        </span>
      </button>

  </div>
</div>

