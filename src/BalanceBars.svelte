<script lang="ts">
  export let balance;

  $: progressBarValues = balances(balance);

  const balances = (balance) => {
    if (Math.abs(balance) <= 100) {
      return [balance];
    }
    let result = new Array(Math.ceil(Math.abs(balance / 100))).fill(100, 1);
    result[0] = Math.abs(balance) % 100 === 0 ? 100 : Math.abs(balance) % 100;
    return result;
  };
</script>

<div class="is-centered box balance-bars is-dark">
  <span class="mb-6" id="balance-label">$ {balance}</span>
  <span
    class="icon is-small js-modal-trigger"
    data-target="settings-modal"
    id="corner-refresh"
  >
    <i class="fas fa-gear" />
  </span>

  {#each progressBarValues as b}
    <progress
      class={`progress is-narrow box ${
        balance > 0 ? "is-primary" : "is-warning"
      }`}
      value={b}
      max="100"
    />
  {/each}
</div>

<style>
  .balance-bars {
    margin-left: 10vw;
    margin-right: 10vw;
    margin-top: 3vh;
    margin-bottom: 3vh;
  }

  #balance-label {
    font-size: x-large;
  }

  #corner-refresh {
    float: right;
  }
</style>
